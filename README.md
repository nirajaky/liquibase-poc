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
