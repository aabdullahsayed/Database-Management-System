# 1. ER Model & Database Design Basics

## Quick Refresher
- **Entity**: a real-world object (e.g., Student, Course).
- **Attribute**: a property of an entity (e.g., Student's name, age).
- **Relationship**: how entities relate (e.g., Student *enrolls in* Course).
- **Cardinality**: how many instances relate — one-to-one, one-to-many, many-to-many.
- **Weak Entity**: an entity that can't be uniquely identified without another entity (e.g., a Room depends on a Building).

## Practice Problems

### Q1 (Basic). What is the difference between an entity and an entity set?
**Answer:** An entity is a single real-world instance (e.g., "John Smith"). An entity set is the collection of all similar entities (e.g., all Students). In a table, the entity set maps to the table, and each entity is a row.

### Q2 (Basic). What is the difference between a strong entity and a weak entity?
**Answer:** A strong entity has its own primary key and exists independently. A weak entity depends on a strong (owner) entity for its identity and typically uses a **partial key** plus the owner's key to be uniquely identified (e.g., "Room 101" only makes sense combined with a specific Building).

### Q3 (Intermediate). Design an ER model for a Library system: Books, Members, and Loans (a member borrows a book on a date and returns it).
**Answer:**
- **Book** (book_id PK, title, author, isbn)
- **Member** (member_id PK, name, email)
- **Loan** (loan_id PK, book_id FK, member_id FK, loan_date, return_date)

Relationship: Member —(borrows, many-to-many via Loan)— Book. The Loan table is a **junction/associative entity** resolving the many-to-many relationship, while also holding relationship attributes (loan_date, return_date).

### Q4 (Intermediate). How do you represent a many-to-many relationship in a relational schema?
**Answer:** You can't represent it directly with a foreign key on either table (that only works for one-to-many). Instead, create a **junction table** with foreign keys to both entities, forming a composite primary key (or a surrogate key), e.g., the `Assignments` table linking Employees and Projects.

### Q5 (Intermediate). What's the difference between total participation and partial participation in ER diagrams?
**Answer:** **Total participation** means every entity instance *must* participate in the relationship (e.g., every Loan must have a Member — shown with a double line). **Partial participation** means participation is optional (e.g., not every Employee manages a Department).

### Q6 (Advanced/Interview). You're designing a schema for a ride-sharing app (like Uber). Identify the core entities, relationships, and one tricky design decision.
**Answer (sample):**
- Entities: `Rider`, `Driver`, `Ride`, `Vehicle`, `Payment`.
- Relationships: Rider *requests* Ride; Driver *accepts* Ride; Driver *owns* Vehicle; Ride *has* Payment.
- **Tricky decision:** Should `Ride` reference `Driver` and `Vehicle` directly, or should it go through an `Assignment` entity to preserve history (since a driver might change vehicles over time, but past rides should still show which vehicle was used)? Best practice: snapshot the vehicle_id **on the Ride record itself** at the time of the ride, rather than always joining live to the Driver's *current* vehicle — this keeps historical data accurate even if the driver later changes cars.

### Q7 (Advanced/Interview). What's the difference between a generalization and specialization in ER modeling? Give an example.
**Answer:** **Specialization** is top-down: start with a general entity (e.g., `Vehicle`) and break it into more specific subtypes (`Car`, `Motorcycle`, `Truck`) with their own unique attributes. **Generalization** is bottom-up: start with specific entities and combine common attributes into a general parent entity. Both usually get implemented in SQL using either a single table with a "type" column, or separate subtype tables linked by a shared key (table-per-subtype inheritance).
