# Design a Hospital System

## Scenario
A hospital needs to track patients, doctors, appointments, and prescriptions - with the constraint that a doctor cannot be double-booked, and prescriptions must reference exactly which appointment they were issued during (for audit/compliance).

## Requirements
- Patients can have many appointments over time, with many different doctors.
- A doctor cannot have two appointments overlapping in time.
- Each prescription is tied to a specific appointment (for audit trail).
- Doctors have specialties; patients may need to filter doctors by specialty.

## Diagram
```
Patient          Appointment            Doctor              Prescription
+---------+  1 M +-------------+  M  1  +--------+   1   M  +--------------+
| pat_id  |----->| appt_id     |<-------| doc_id |          | prescr_id    |
| name    |      | patient_id  |        | name   |          | appt_id FK   |
+---------+      | doctor_id   |        | spec.  |          | medication   |
                 | start_time  |        +--------+          | dosage       |
                 | end_time    |                             +--------------+
                 +-------------+
```

## Schema
```sql
CREATE TABLE Patient (
    patient_id SERIAL PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    dob        DATE NOT NULL
);

CREATE TABLE Doctor (
    doctor_id SERIAL PRIMARY KEY,
    name      VARCHAR(100) NOT NULL,
    specialty VARCHAR(100)
);

CREATE TABLE Appointment (
    appt_id    SERIAL PRIMARY KEY,
    patient_id INT NOT NULL REFERENCES Patient(patient_id),
    doctor_id  INT NOT NULL REFERENCES Doctor(doctor_id),
    start_time TIMESTAMPTZ NOT NULL,
    end_time   TIMESTAMPTZ NOT NULL,
    CHECK (end_time > start_time)
    -- overlap prevention: EXCLUDE USING gist (doctor_id WITH =, tsrange(start_time,end_time) WITH &&)
);

CREATE TABLE Prescription (
    prescr_id  SERIAL PRIMARY KEY,
    appt_id    INT NOT NULL REFERENCES Appointment(appt_id),
    medication VARCHAR(150) NOT NULL,
    dosage     VARCHAR(100) NOT NULL,
    issued_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Key design decisions
- The "no double-booking a doctor" rule is a **range-overlap** constraint, not expressible with a simple `UNIQUE` constraint - Postgres's `EXCLUDE USING gist` constraint (with the `btree_gist` extension) is purpose-built for exactly this: "no two rows for the same doctor_id with overlapping time ranges."
- `Prescription` references `appt_id`, not `patient_id` directly - this preserves the audit requirement ("which specific visit led to this prescription") rather than just loosely linking patient-to-medication.
- `specialty` on `Doctor` is a plain column here for simplicity; at larger scale (doctors with multiple specialties), it would become its own `DoctorSpecialty` junction table (many-to-many).

## Takeaway
Real-world "no double-booking" constraints are range-overlap problems, not simple uniqueness problems - know that your database likely has a dedicated feature for this (exclusion constraints) rather than trying to hack it with application-level checks alone, which are race-prone under concurrency.
