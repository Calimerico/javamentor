## How to introduce Liquibase in your project?

### Goals of the article

1. We will show you how to take a snapshot from existing database
2. Generate new migration script for every small change after initial snapshot(whenever you add column, add table etc.) 
3. How to generate migration scripts for small changes based on change in hibernate entity classes(with maven/gradle plugin)
4. Applying those migration scripts to the database

### Install and configure liquibase

If you work without help of maven/gradle plugin you need to install `liquibase`. Why would you work without one of those plugins?
Well, maybe you are not familiar with maven/gradle(you are devops for example) or you don't have access for source code.

Command for installing `liquibase` : `sudo snap install liquibase`
Now just download [this](https://www.liquibase.org/download) and extract.


You will notice that in root of extracted folder there is `lib` folder. In that `lib` folder you need to add `.jar` of your database driver.


Navigate to `examples/xml` folder. (You can also navigate to to `sql` folder and work with that king of changes but in this tutorial we will cover `xml` way)
In `examples/xml`, you will find `liquibase.properties`.

Now, you can either change values of properties in this `liquibase.properties` file or pass those properties through command line(what's easier for you).
In this case, let's change the file.
Open that file and change next properties: `url`, `username`, `password`, `referenceUrl`, `referenceUsername`, `referencePassword`.
Also, you need to add `classpath: ../../lib/<YOUR-DRIVER>.jar` . Make sure that you replace `<YOUR-DRIVER>.jar` with name of the real driver(you added your `.jar` inside `lib` folder, didn't you).

### Get database snapshot and apllying that snapshot in another db

Execute `liquibase --changeLogFile=snapshot.xml generateChangeLog` and you will get snapshot of your database schema inside `snapshot.xml` file.

As you see, this time we didn't changed `changeLogFile` property but passed it through params of the command.

Before applying this `snapshot.xml` to some other db, you need to specify `url`, `username`, `password`, `referenceUrl`, `referenceUsername`, `referencePassword`
for other db either through file or command params.

Now, you can execute `liquibase --changeLogFile=snapshot.xml update` and that's it, your changes are applied to some other database(one which you specified). 

### Creating migration script for some small change

Now, let's suppose you just add one new column `columnnew` on one of your existing tables `tableold` in db. Your changeset `newChange.xml` looks like this:

```
<?xml version="1.1" encoding="UTF-8" standalone="no"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext" xmlns:pro="http://www.liquibase.org/xml/ns/pro" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd http://www.liquibase.org/xml/ns/pro http://www.liquibase.org/xml/ns/pro/liquibase-pro-4.0.xsd http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.0.xsd">
    <changeSet author="spasoje" id="1622635456892-2">
        <addColumn tableName="tableold">
            <column name="columnnew" type="uuid">
                <constraints nullable="true"/>
            </column>
        </addColumn>
    </changeSet>
</databaseChangeLog>

```
And now execute `liquibase --changeLogFile=newChange.xml update` and you added new column.

### Executing list of scripts

Now, if you want to apply both `snapshot.xml` and `newChange.xml` in one go against some empty db you can do it in this way. Create `master.xml` with this content:

```
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <include file="snapshot.xml" relativeToChangelogFile="true"/>
    <include file="newChange.xml" relativeToChangelogFile="true"/>

</databaseChangeLog>
```

Execute `liquibase --changeLogFile=master.xml update` and that's it you have both scripts applied to the db.

### Generating migration scripts with gradle/maven based on hibernate entities

#### With gradle

You need to add a gradle plugin and to specify few things. `build.gradle` should look something like this(you can extract parts related to the liquidbase):

```
import java.text.SimpleDateFormat

buildscript {
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath "org.liquibase:liquibase-gradle-plugin:2.0.3"
	}
}

plugins {
	id 'org.springframework.boot' version '2.5.0'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
	id 'org.liquibase.gradle' version '2.0.3'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	liquibaseRuntime.extendsFrom runtime
	liquibaseRuntime.extendsFrom implementation
}

repositories {
	mavenCentral()
}


apply plugin: 'org.liquibase.gradle'


liquibase {
	activities {
		main {
			def df = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm'Z'") // you can change it
			df.setTimeZone(TimeZone.getTimeZone("UTC"))
			changeLogFile "src/main/db/mainDiff-${df.format(new Date())}.xml"
			url 'jdbc:postgresql://localhost:5432/db'
			username 'postgres'
			password 'changeme'
			referenceUrl 'hibernate:spring:com.example?dialect=org.hibernate.dialect.PostgreSQL94Dialect&hibernate.implicit_naming_strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyComponentPathImpl&hibernate.physical_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy'
			referenceDriver 'liquibase.ext.hibernate.database.connection.HibernateDriver'
		}
	}
	runList = 'main'
}

diff.dependsOn compileJava
diffChangeLog.dependsOn compileJava
generateChangelog.dependsOn compileJava

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'org.postgresql:postgresql'
	liquibaseRuntime 'org.postgresql:postgresql'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	liquibaseRuntime 'org.liquibase:liquibase-core:3.8.1'
	liquibaseRuntime 'org.liquibase:liquibase-groovy-dsl:2.1.1'
	liquibaseRuntime 'mysql:mysql-connector-java:5.1.34'
	liquibaseRuntime group: 'javax.xml.bind', name: 'jaxb-api', version: '2.3.1'
	liquibaseRuntime("ch.qos.logback:logback-core:1.2.3")
	liquibaseRuntime("ch.qos.logback:logback-classic:1.2.3")
	liquibaseRuntime 'org.liquibase.ext:liquibase-hibernate5:3.6'
	liquibaseRuntime sourceSets.main.output

}
```
Of course, you need to change things inside `main` activity to match your url, db driver etc. Don't forget to change `com.example` to the package where your hibernate entities reside.

Now, you can run this command: `./gradlew diffChangeLog -PrunList=main` and this command will give you the difference between database schema and hibernate entities.

Command `./gradlew generateChangelog` will generate db snapshot for you.

Command `./gradlew update` will update your db with `changeLogFile` specified inside `build.gradle` file. Also you can pass `changeLogFile` through gradle plugin parameter.

### Maven plugin

The similar thing that can be done with gradle, can be done through maven. You just need one plugin.

```
<plugin>
	<groupId>org.liquibase</groupId>
	<artifactId>liquibase-maven-plugin</artifactId>
	<version>4.0.0</version>
	<configuration>
		<diffChangeLogFile>${diffChangeLogFile}</diffChangeLogFile>
		<propertyFile>src/main/resources/liquibase.properties</propertyFile>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>org.liquibase.ext</groupId>
			<artifactId>liquibase-hibernate5</artifactId>
			<version>4.0.0</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
			<version>${project.parent.version}</version>
		</dependency>
		<dependency>
			<groupId>javax.validation</groupId>
			<artifactId>validation-api</artifactId>
			<version>2.0.1.Final</version>
		</dependency>
	</dependencies>
</plugin>
```
Also, with maven plugin you need this `liquibase.properties`:

```
url=jdbc:postgresql://localhost:5432/db
username=postgres
password=changeme
driver=org.postgresql.Driver
changeLogFile=src/main/resources/db/changelog-master.xml
diffExcludeObjects=table:act_.*

referenceUrl=hibernate:spring:com.example?dialect=org.hibernate.dialect.PostgreSQL94Dialect&hibernate.implicit_naming_strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyComponentPathImpl&hibernate.physical_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
referenceDriver=liquibase.ext.hibernate.database.connection.HibernateDriver
```
For sure, change this file to your needs.

Now, difference between db and hibernate can be generated by: `mvn clean install -Dmaven.test.skip=true liquibase:diff`

As you might guess, generating db snapshot with plugin is straightforward: `mvn clean install -Dmaven.test.skip=true liquibase:generateChangeLog`

Applying changes to db: `mvn clean install -Dmaven.test.skip=true --changeLogFile=liquibase:update` (you just need to specify `changeLogFile` through params or by changing liquibase.properties)
