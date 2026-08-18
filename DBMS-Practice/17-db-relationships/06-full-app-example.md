# 6. Full Application Example: A Blog Platform

Let's design one realistic app and use all three relationship shapes together — the way they actually show up in a real backend.

## The scenario

- Each **user** has exactly one **profile** (1:1 — bio, avatar, rarely queried together with login).
- Each **user** can write many **posts**; each post has exactly one author (1:N).
- Each **post** can have many **tags**, and each **tag** can apply to many posts (N:M).
- Each **post** can have many **comments**; each comment belongs to one post and one commenting user (two separate 1:N relationships).

## The full picture

```
users ──1:1── user_profiles

users ──1:N── posts ──N:M── tags   (via post_tags junction table)
                │
                └──1:N── comments ──N:1── users   (commenter)
```

## Raw SQL schema

```sql
CREATE TABLE users (
    id       SERIAL PRIMARY KEY,
    email    TEXT UNIQUE NOT NULL,
    name     TEXT NOT NULL
);

-- 1:1 with users
CREATE TABLE user_profiles (
    id       SERIAL PRIMARY KEY,
    user_id  INTEGER UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    bio      TEXT,
    avatar_url TEXT
);

-- 1:N with users (FK on the "many" side)
CREATE TABLE posts (
    id        SERIAL PRIMARY KEY,
    title     TEXT NOT NULL,
    body      TEXT,
    author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE tags (
    id    SERIAL PRIMARY KEY,
    name  TEXT UNIQUE NOT NULL
);

-- N:M junction table between posts and tags
CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    tag_id  INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

-- 1:N with posts AND 1:N with users (two foreign keys, two relationships)
CREATE TABLE comments (
    id         SERIAL PRIMARY KEY,
    post_id    INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    user_id    INTEGER REFERENCES users(id) ON DELETE CASCADE,
    body       TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
);
```

Notice `comments` has **two** foreign keys — that's normal. A row can participate in multiple relationships at once; each FK is a separate "arrow" pointing to a separate "one" side.

## The same schema in SQLAlchemy

```python
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True)
    name = Column(String)

    profile = relationship("UserProfile", back_populates="user", uselist=False)
    posts = relationship("Post", back_populates="author")
    comments = relationship("Comment", back_populates="user")


class UserProfile(Base):
    __tablename__ = "user_profiles"
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"), unique=True)
    bio = Column(Text)

    user = relationship("User", back_populates="profile")


post_tags = Table(
    "post_tags", Base.metadata,
    Column("post_id", ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", ForeignKey("tags.id"), primary_key=True),
)


class Post(Base):
    __tablename__ = "posts"
    id = Column(Integer, primary_key=True)
    title = Column(String)
    author_id = Column(Integer, ForeignKey("users.id"))

    author = relationship("User", back_populates="posts")
    tags = relationship("Tag", secondary=post_tags, back_populates="posts")
    comments = relationship("Comment", back_populates="post")


class Tag(Base):
    __tablename__ = "tags"
    id = Column(Integer, primary_key=True)
    name = Column(String, unique=True)

    posts = relationship("Post", secondary=post_tags, back_populates="tags")


class Comment(Base):
    __tablename__ = "comments"
    id = Column(Integer, primary_key=True)
    post_id = Column(Integer, ForeignKey("posts.id"))
    user_id = Column(Integer, ForeignKey("users.id"))

    post = relationship("Post", back_populates="comments")
    user = relationship("User", back_populates="comments")
```

## What you get from the ORM: navigable objects instead of manual joins

```python
alice = session.query(User).filter_by(email="alice@x.com").first()

alice.profile.bio                     # 1:1 — no join written by hand
[p.title for p in alice.posts]         # 1:N — no join written by hand
alice.posts[0].tags                    # N:M — no junction table SQL written by hand
alice.posts[0].comments[0].user.name   # chains across multiple relationships
```

Every one of those lines quietly becomes a `JOIN` behind the scenes — the same SQL you'd write by hand, generated for you because you told the ORM, once, what shape each relationship is.

## A note on N+1 queries — the most common ORM performance bug

If you loop over posts and access `post.author.name` inside the loop, a naive ORM setup issues **one separate query per post** to fetch each author — this is the infamous "N+1 query problem." It's not a flaw in the relationship model, it's a *loading strategy* problem. Fix it by telling the ORM to fetch related data eagerly, in one query:

```python
# SQLAlchemy: eager-load authors in the same query
posts = session.query(Post).options(joinedload(Post.author)).all()
```
```python
# Django: same idea
posts = Post.objects.select_related("author").all()          # for 1:N / 1:1 (FK on this table)
posts = Post.objects.prefetch_related("tags").all()           # for N:M / reverse FK
```
```typescript
// Prisma: include related data in the same query
const posts = await prisma.post.findMany({ include: { author: true, tags: true } });
```

This is worth internalizing early: **knowing the relationship shape tells you how to model the schema; knowing about eager vs. lazy loading tells you how to query it efficiently.** Both matter, and they're separate skills.

## Recap: the whole guide in one table

| Question to ask | Shape | Tell |
|---|---|---|
| Can each side only ever point to one row on the other side? | 1:1 | `UNIQUE` foreign key |
| Does one row have many related rows, but each related row has only one owner? | 1:N | Plain foreign key on the "many" side |
| Can both sides have many matches with each other? | N:M | A junction table with two foreign keys |

Every relationship you'll ever model — in raw SQL or through any ORM — is one of these three answers.
