### SchemaManagementException: Export identifier [...] encountered more than once

# Problem

Let's say we have those two classes:
```
@Entity
public class Foo {

        @Id
        private UUID id = UUID.randomUUID();
    
        @OneToMany
        
        private List<FooDetails> details;
    
        public void setDetails(List<FooDetails> details) {
            this.details = details;
        }
    
        public UUID getId() {
            return id;
        }
    
        public List<> getDetails() {
            return details;
        }
}
```
and
```
@Entity
public class FooDetails {

        @Id
        private UUID id;
        private String name;
    
        ...getters and setters
}
```
If we try to save Foo, we will receive an exception(I am pasting just relevant part):
```
Caused by: org.hibernate.tool.schema.spi.SchemaManagementException: Export identifier [foo_details] encountered more than once
	at org.hibernate.tool.schema.internal.AbstractSchemaMigrator.checkExportIdentifier(AbstractSchemaMigrator.java:487) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.tool.schema.internal.GroupedSchemaMigratorImpl.performTablesMigration(GroupedSchemaMigratorImpl.java:68) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.tool.schema.internal.AbstractSchemaMigrator.performMigration(AbstractSchemaMigrator.java:207) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.tool.schema.internal.AbstractSchemaMigrator.doMigration(AbstractSchemaMigrator.java:114) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.tool.schema.spi.SchemaManagementToolCoordinator.performDatabaseAction(SchemaManagementToolCoordinator.java:184) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.tool.schema.spi.SchemaManagementToolCoordinator.process(SchemaManagementToolCoordinator.java:73) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.internal.SessionFactoryImpl.<init>(SessionFactoryImpl.java:320) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.boot.internal.SessionFactoryBuilderImpl.build(SessionFactoryBuilderImpl.java:462) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl.build(EntityManagerFactoryBuilderImpl.java:1237) ~[hibernate-core-5.4.9.Final.jar:5.4.9.Final]
	at org.springframework.orm.jpa.vendor.SpringHibernateJpaPersistenceProvider.createContainerEntityManagerFactory(SpringHibernateJpaPersistenceProvider.java:58) ~[spring-orm-5.2.2.RELEASE.jar:5.2.2.RELEASE]
	at org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean.createNativeEntityManagerFactory(LocalContainerEntityManagerFactoryBean.java:365) ~[spring-orm-5.2.2.RELEASE.jar:5.2.2.RELEASE]
	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.buildNativeEntityManagerFactory(AbstractEntityManagerFactoryBean.java:391) ~[spring-orm-5.2.2.RELEASE.jar:5.2.2.RELEASE]
	... 20 common frames omitted
```
As you can see <a href="https://stackoverflow.com/questions/42630661/sql-strings-added-more-than-once-for-tablename">here</a> problem is that more than one entity start with the same prefix(in our case `Foo` and `FooDetails`)
If we just rename `FooDetails` to `Details`, it will work perfectly.
