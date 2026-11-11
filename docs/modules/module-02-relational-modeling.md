# Module 2 — Relational Modeling, Keys and Normalization

## Outcomes

- Select primary and candidate keys based on domain identity.
- Enforce uniqueness, nullability, and referential integrity.
- Explain functional dependencies and 1NF, 2NF, 3NF and BCNF.
- Model 1:1, 1:N and M:N relationships.
- Justify denormalization without losing track of consistency.

## 1. Keys and constraints

- Primary key: the selected candidate key that identifies a row; one PK constraint per table, possibly composite.
- Candidate key: a minimal set of attributes that uniquely identifies a row.
- Natural key: meaningful in the business domain, such as a SKU; may be mutable or carry external rules.
- Surrogate key: a generated or artificial identifier separated from business attributes.
- Composite key: identity made from multiple columns.
- Foreign key: validates references to a candidate/primary key in another or the same table.
- UNIQUE, NOT NULL and CHECK constraints express additional data rules.

Example:

    CREATE TABLE customers (
        customer_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        email VARCHAR(320) NOT NULL UNIQUE,
        name VARCHAR(150) NOT NULL
    );

    CREATE TABLE orders (
        order_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        customer_id BIGINT NOT NULL REFERENCES customers(customer_id),
        created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
        status VARCHAR(20) NOT NULL
    );

A foreign key does not imply NOT NULL. A nullable FK permits NULL values unless NOT NULL is also declared. PostgreSQL allows multiple NULLs in a standard UNIQUE constraint by default; database semantics can differ.

## 2. Functional dependencies

X → Y means X determines Y according to the rules of the modeled domain.

Examples:
- customer_id → customer_name
- product_id → product_name
- (order_id, line_number) → quantity and product_id

Functional dependencies come from business rules, not just a sample dataset. They help identify attributes stored in the wrong relation.

## 3. Normal forms

### 1NF

Model each attribute at the intended atomic level and avoid repeating groups hidden inside a single value. For example, do not put a comma-separated list of phone numbers in the customer row if the numbers need independent validation and querying. Create a customer_phones relation instead.

This does not mean arrays or JSON are always wrong; choose them based on the access pattern and whether individual members need relational constraints.

### 2NF

Be in 1NF and avoid non-prime attributes depending on only part of a candidate key. If (order_id, product_id) is a key but product_name depends only on product_id, move product_name to products. The quantity in a line item depends on the order/product relationship.

### 3NF

Be in 2NF and avoid certain transitive dependencies of non-key attributes on a candidate key. For example, employee_id → department_id and department_id → department_name suggests department_name belongs to departments.

### BCNF

For every non-trivial functional dependency X → Y, X must be a superkey. BCNF is stricter than 3NF in some dependency patterns. Be ready to analyze a small dependency example rather than merely recite the acronym.

## 4. Common anomalies

- Update anomaly: the same fact appears repeatedly and only some copies are updated.
- Insert anomaly: one fact cannot be inserted without inventing an unrelated fact.
- Delete anomaly: deleting one record accidentally removes the only copy of another fact.

Normalization reduces these kinds of redundancy-related problems. It does not prohibit deliberate snapshots or read models.

## 5. Relationship cardinality and optionality

- 1:1: a foreign key plus a UNIQUE constraint can enforce one-to-one association; consider whether both entities need separate tables.
- 1:N: put the FK on the many-side in the normal relational design, e.g. orders.customer_id.
- M:N: use a junction table such as enrollments(student_id, course_id, enrolled_at).

    CREATE TABLE enrollments (
        student_id BIGINT NOT NULL REFERENCES students(student_id),
        course_id BIGINT NOT NULL REFERENCES courses(course_id),
        enrolled_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (student_id, course_id)
    );

If the domain allows multiple enrollments over different terms, the key must include the term or enrollment identity and enforce the intended uniqueness rule.

## 6. Foreign-key delete behavior

- RESTRICT / NO ACTION: reject deletes that violate references, subject to constraint timing.
- CASCADE: delete dependent rows.
- SET NULL: clear the reference if the FK column is nullable.

Choose actions from lifecycle semantics, audit requirements and retention rules. Cascading deletion of financial or audit history is often inappropriate.

## 7. Denormalization and snapshots

An order item should usually preserve the unit price actually charged at purchase time. The product catalogue price may change later; the historical transaction must remain interpretable. Other transaction snapshots can include product name, discount, tax, currency and fulfillment terms as required.

A stored customer_order_count may improve read latency, but every create/cancel/delete/retry path must maintain it correctly or reconcile drift. Define the source of truth and acceptable staleness.

## Schema design checklist

- State each table's row grain.
- Identify candidate keys and choose a primary key.
- Declare required versus optional attributes.
- Enforce appropriate uniqueness, FK and CHECK constraints.
- Model M:N relationships explicitly.
- Identify historical facts that must remain snapshots.
- Review deletion semantics and retention requirements.
- Use indexes to support real queries; remember PostgreSQL does not automatically index the referencing side of every foreign key.

## References

- [PostgreSQL — Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [PostgreSQL — Data definition](https://www.postgresql.org/docs/current/ddl.html)
- [IBM — Database normalization](https://www.ibm.com/think/topics/database-normalization)
- [MongoDB — Data modeling](https://www.mongodb.com/docs/manual/data-modeling/)
