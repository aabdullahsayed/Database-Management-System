# 5. How ORMs Express These Relationships

## The core idea: the ORM is a translator, not a new concept

Every ORM below is describing the *exact same three shapes* from files 2–4. The syntax differs, the underlying SQL (foreign keys, `UNIQUE` constraints, junction tables) does not. Once you can look at any ORM snippet and mentally translate it back to "which table holds the foreign key, and is there a UNIQUE constraint," every ORM becomes readable — even ones you've never used.

Think of it like learning that "hola," "bonjour," and "hello" all point at the same greeting. The ORMs are just different accents for "this row points to that row."

---

## 5.1 One-to-One

### SQLAlchemy (Python)
```python
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    profile = relationship("UserProfile", back_populates="user", uselist=False)

class UserProfile(Base):
    __tablename__ = "user_profiles"
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"), unique=True)  # <- the UNIQUE from file 2
    user = relationship("User", back_populates="profile")
```
`uselist=False` is SQLAlchemy telling you "don't treat this as a list — there's only ever one." That's the ORM's way of encoding the passport analogy.

### Django ORM (Python)
```python
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()
```
Django just names it what it is: `OneToOneField`. Under the hood, this generates a `UNIQUE` foreign key column — same as SQLAlchemy, same as the raw SQL in file 2.

### Prisma (TypeScript/Node)
```prisma
model User {
  id      Int          @id @default(autoincrement())
  profile UserProfile?
}

model UserProfile {
  id     Int  @id @default(autoincrement())
  userId Int  @unique
  user   User @relation(fields: [userId], references: [id])
}
```
The `@unique` on `userId` is doing the same job as `unique=True` in Django and `UNIQUE` in raw SQL — it's the constraint that makes this 1:1 instead of 1:N.

### Mongoose (MongoDB, Node)
MongoDB has no foreign keys, so 1:1 is usually modeled by **embedding** rather than referencing:
```js
const userSchema = new Schema({
  name: String,
  profile: { bio: String, avatarUrl: String }  // embedded directly, not a separate collection
});
```
This is the document-database equivalent of the passport being stapled directly inside the person's file rather than kept in a separate drawer — reasonable *because* it's strictly 1:1 and rarely needs independent querying.

---

## 5.2 One-to-Many

### SQLAlchemy
```python
class User(Base):
    id = Column(Integer, primary_key=True)
    posts = relationship("Post", back_populates="author")   # the "many" side, plural name

class Post(Base):
    id = Column(Integer, primary_key=True)
    author_id = Column(Integer, ForeignKey("users.id"))       # FK lives here, on the "many" side
    author = relationship("User", back_populates="posts")
```
Notice `posts` (plural, on `User`) vs. `author` (singular, on `Post`) — the naming itself encodes "one teacher, many students" from file 3.

### Django ORM
```python
class Post(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name="posts")
```
One line. Django infers the whole 1:N shape from a single `ForeignKey` because — as file 3 explained — the FK *always* lives on the "many" side, so there's no ambiguity to configure. `related_name="posts"` is what lets you write `alice.posts.all()` from the "one" side.

### Prisma
```prisma
model User {
  id    Int    @id @default(autoincrement())
  posts Post[]                          // array = the "many" side
}

model Post {
  id       Int  @id @default(autoincrement())
  authorId Int
  author   User @relation(fields: [authorId], references: [id])
}
```
`Post[]` (an array type) is Prisma's way of visually showing "many" right in the schema — a nice readability win over SQLAlchemy/Django where you have to know the convention.

### Mongoose
```js
const postSchema = new Schema({
  title: String,
  authorId: { type: Schema.Types.ObjectId, ref: "User" }
});

// fetch a user's posts:
const posts = await Post.find({ authorId: someUserId });
```
Same shape as SQL — a reference on the "many" document pointing back to the "one" document — just without a formal foreign-key constraint enforced by the database itself (MongoDB won't stop you from referencing a deleted user; your application code has to handle that).

---

## 5.3 Many-to-Many

### SQLAlchemy
```python
enrollments = Table(
    "enrollments", Base.metadata,
    Column("student_id", ForeignKey("students.id"), primary_key=True),
    Column("course_id", ForeignKey("courses.id"), primary_key=True),
)

class Student(Base):
    id = Column(Integer, primary_key=True)
    courses = relationship("Course", secondary=enrollments, back_populates="students")

class Course(Base):
    id = Column(Integer, primary_key=True)
    students = relationship("Student", secondary=enrollments, back_populates="courses")
```
`secondary=enrollments` is SQLAlchemy explicitly pointing at the junction table from file 4. This is the "simple" form — use it only when the junction table needs no extra columns of its own.

If the junction table needs extra data (like `enrolled_on` from file 4), you model it as its own full class instead of a bare `Table`, with two separate one-to-many relationships into it — exactly the "two 1:N relationships into a junction table" diagram from file 4.

### Django ORM
```python
class Course(models.Model):
    students = models.ManyToManyField(Student, through="Enrollment", related_name="courses")

class Enrollment(models.Model):     # the junction table, made explicit because it carries data
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_on = models.DateField(auto_now_add=True)
```
`through="Enrollment"` is Django saying "don't auto-generate a hidden junction table — use this explicit one," which you need the moment the junction table carries its own data (the "seating chart with dietary preferences" idea from file 4).

### Prisma
```prisma
model Student {
  id          Int          @id @default(autoincrement())
  enrollments Enrollment[]
}

model Course {
  id          Int          @id @default(autoincrement())
  enrollments Enrollment[]
}

model Enrollment {
  studentId  Int
  courseId   Int
  enrolledOn DateTime  @default(now())
  student    Student   @relation(fields: [studentId], references: [id])
  course     Course    @relation(fields: [courseId], references: [id])
  @@id([studentId, courseId])
}
```
Prisma always makes the junction table explicit as its own model — there's no "hidden" shorthand once data needs to live on the pairing, which keeps the schema honest about what's really happening underneath.

### Mongoose
Two common patterns:
```js
// Pattern A: array of references on both sides (fine for small, rarely-changing lists)
const studentSchema = new Schema({
  name: String,
  courseIds: [{ type: Schema.Types.ObjectId, ref: "Course" }]
});

// Pattern B: an explicit junction collection (better when the pairing carries data,
// or when either side's list could grow large)
const enrollmentSchema = new Schema({
  studentId: { type: Schema.Types.ObjectId, ref: "Student" },
  courseId: { type: Schema.Types.ObjectId, ref: "Course" },
  enrolledOn: { type: Date, default: Date.now }
});
```

---

## 5.4 The pattern across all four ORMs

| Concept from files 2-4 | SQLAlchemy | Django | Prisma | Mongoose |
|---|---|---|---|---|
| FK with UNIQUE (1:1) | `uselist=False` | `OneToOneField` | `@unique` on FK field | embedded subdocument |
| FK on the "many" side (1:N) | `relationship(...)` + `ForeignKey` | `ForeignKey` | array field (`Post[]`) | ref field + manual query |
| Junction table (N:M) | `secondary=` table or explicit class | `ManyToManyField(through=...)` | explicit junction model | array of refs or junction collection |

Once you can fill in this table from memory, you understand ORM relationships — the vocabulary is the only thing that changes between frameworks.

Next: [`06-full-app-example.md`](06-full-app-example.md) — a complete blog-platform schema using all three shapes together, in SQL and in code.
