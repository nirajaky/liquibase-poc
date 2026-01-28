# liquibase-poc
* Liquibase is a database management tool. That keeps the track of any changes in our DB
* We add the changes/alter in DB in our change-log
* We can migrate our DB with liquibase as well

## Configure liquibase in our Application (3 Ways)
* xml-based config (old)
* yaml based (use change-log and change-set)
* using .sql based

## Configure (.sql based)
1. Add liquibase-core in our pom.xml
2. Added liquibase plugin in POM with path - where we want to keep our liquibase properties.
   * If you don't add this plugin:
      - We cannot run Liquibase commands via Maven ( eg. mvn liquibase:update)
      - Database migrations must be triggered by our application at startup (using liquibase-core or spring-boot-starter-liquibase).
      - We lose the ability to manage, validate, or rollback database changes independently of our app.
4. Then we created liquibase.properties with DB configuration and also, provided the path where the changelog should be triggered.
5. Then we can run-> mvn liquibase:update (It checks any liquibase and run for once)
6. Note: In .sql file even comments are read. Whereas, In JAVA - java compiler ignores the comment.

## Changelog tables
1. databasechangelog: It logs(or keep the data) of all the changes that where executed in the DB with info like date time author, descriptions etc.
2. databasechangeloglock: It creates a lock to the DB. So, that only 1 transactions happens to DB whenever liquibase is running.
    - Liquibase acquires a lock before running migrations and releases it after completion. This table is automatically created and managed by Liquibase.
    - When Liquibase acquires a lock (using the databasechangeloglock table), it prevents other Liquibase processes from running migrations. However, regular application queries (such as SELECT, INSERT, UPDATE, DELETE) are not blocked by this lock. Only Liquibase migration operations wait for the lock to be released; normal app database operations continue unaffected.
  
## Avoid any checksum error
- If this DB is shared / prod-like:
- Revert the SQL file to original
- Create a new changeset instead
- Liquibase rule:
  * Never modify an executed changeset
- Temp Fix :
  * **mvn liquibase:clearCheckSums
mvn liquibase:update
**
- Commands to generate the schema of cureent connected DB:
  *  mvn liquibase:generateChangeLog "-Dliquibase.outputChangeLogFile=src/main/resources/db/changelog/existingDbLog.sql"
 
## “How do professionals prevent JPA entities and Liquibase SQL from drifting apart?”

Short answer: you NEVER let Hibernate change prod schema, and you NEVER trust humans alone to keep them in sync. You add checks.

Let’s go step-by-step, the right way.

🧠 Golden Rule (print this)

Entities describe how Java uses the DB.
Liquibase defines how the DB actually changes.
Hibernate must NEVER auto-update prod.

🎯 Your scenario

You want to:

Change column length

Add a @OneToMany relationship

And everything is already in production.

✅ Correct workflow (industry standard)
1️⃣ Change the ENTITY (always first)

Example:

@Entity
@Table(name = "employee")
public class Employee {

    @Column(name = "name", length = 200) // changed from 100 → 200
    private String name;

    @OneToMany(mappedBy = "employee", fetch = FetchType.LAZY)
    private List<Task> tasks;
}


And new entity:

@Entity
@Table(name = "task")
public class Task {

    @ManyToOne
    @JoinColumn(name = "employee_id", nullable = false)
    private Employee employee;
}


👉 This expresses intent
👉 But does NOT change DB

2️⃣ Write Liquibase SQL changeset (source of truth)

Create new migration file (never modify old ones):

003-alter-employee-add-task.sql

-- liquibase formatted sql

-- changeset niraj:003-employee-name-length
ALTER TABLE employee
ALTER COLUMN name TYPE VARCHAR(200);

-- changeset niraj:004-create-task-table
CREATE TABLE task (
    id BIGSERIAL PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    description VARCHAR(255),
    CONSTRAINT fk_task_employee
      FOREIGN KEY (employee_id) REFERENCES employee(id)
);


🚫 Do NOT rely on Hibernate
🚫 Do NOT generate SQL automatically

🔥 Now the BIG question:
How do we validate entity ↔ DB sync?

This is where most teams fail.

🛡️ 5 PROVEN ways to prevent mismatch
🥇 Hibernate schema validation (MANDATORY)

In non-prod environments:

spring:
  jpa:
    hibernate:
      ddl-auto: validate


What this does:

App fails to start if:

Column length mismatch

Missing FK

Wrong nullability

Wrong column type

❌ It will catch:

Wrong VARCHAR length

Missing employee_id

Missing FK

✅ This is your first safety net

🥈 Integration test using real DB

Use Testcontainers:

@Testcontainers
@SpringBootTest
class SchemaValidationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16");

}


Liquibase runs → Hibernate validates → test passes/fails

💥 CI fails if SQL ≠ entity

🥉 Liquibase diff (manual guardrail)

Compare entity-generated schema vs DB:

mvn liquibase:diff


What you check:

Unexpected diffs = 🔥 red flag

Missing FK

Length mismatch

⚠️ Never auto-apply diffs
Use them as review tool

🏅 Code review rule (human layer)

Every PR that touches:

@Entity

@Column

relationship annotations

Must also contain:

New Liquibase migration file

No migration → PR rejected ❌

🧯 Prod safety switch

In production:

spring:
  jpa:
    hibernate:
      ddl-auto: none


Hibernate can:

Read

Validate

BUT NEVER modify

🚨 What NOT to do (very common mistakes)

❌ Modify old Liquibase files
❌ Use ddl-auto=update in prod
❌ Let Hibernate generate SQL
❌ Skip FK constraints
❌ Trust “it works on my machine”

🧩 Relationship changes — subtle but important
@OneToMany does NOT create columns

👉 @ManyToOne side does

So you must:

Add FK column manually in SQL

Add constraint explicitly

Hibernate will NOT fix missing FKs.

🧪 Extra: catching subtle mistakes
Mistake	Who catches it
Wrong column length	Hibernate validate
Missing FK	Hibernate validate
Wrong nullable	Hibernate validate
Missing index	❌ Manual
Performance issue	❌ DBA / review
🏆 Ideal production setup (TL;DR)
Entity change
   ↓
Liquibase SQL migration
   ↓
CI runs Liquibase
   ↓
Hibernate ddl-auto=validate
   ↓
App starts or FAILS

💬 Real talk

Liquibase defines history
Hibernate validates correctness

That’s the clean separation.
