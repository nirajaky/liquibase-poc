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
