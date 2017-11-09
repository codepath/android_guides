![Alt text](http://www.softwaretree.com/v1/images/jdx-anroide-logo.png "JDXA, The KISS ORM for Android")

# An Introduction


This is a brief introduction to the [JDXA](http://www.softwaretree.com/v1/products/jdxa/jdxa.html) Object Relational Mapping (ORM) product. JDXA (a.k.a. JDX for Android) can be used to easily develop mobile apps that require intuitive object-oriented access to device-local SQLite relational data on the Android platform. This document provides an overview of JDXA followed by brief instructions on how to install and use the product. It also provides a simple code example and descriptions of some Android-specific utility classes to simplify app development with JDXA.

This document also contains links to some illustrative code snippets for different mapping scenarios like one-to-one, one-to-many, and many-to-many relationships, and JSON object persistence.  Finally, there is a list of many sample projects shipped with the product SDK that demonstrate how different features of JDXA may easily be used to simplify object modeling and data integration tasks for Android apps.

##### [**Download JDXA ORM SDK**](http://softwaretree.com/v1/products/jdxa/download-jdxa.php)

The SDK comes with a comprehensive user manual, javadocs, and many working sample apps.

**Note:** JDX is the core Object Relational Mapping (ORM) technology developed by Software Tree. This core ORM technology has been adapted for the Android platform and that adaptation is called JDXA. In the context of the Android platform, the terms _JDX_, _JDXA_, and _JDX for Android_ may be used synonymously.

## **Overview**

In Object Oriented Programming (OOP) languages (e.g. Java, C++, C#), a class encapsulates the structure and behavior of objects of a certain type.  Business objects are easier to represent as instances of classes.  A domain object model consists of various classes and their relationships comprising an application domain. Integration of domain object model data with relational data is a common need for most object-oriented applications. ORM has become the preferred and most popular paradigm for such integration. Our JDXA ORM product (a.k.a. JDX for Android) increases programmer productivity tremendously by presenting a more intuitive, object-oriented view of relational data and eliminating the need to write endless lines of complex low-level SQL code. Based on some well thought-out [KISS (Keep It Simple and Straightforward) principles](http://www.softwaretree.com/products/njdx/whitepaper/KISSPrinciples.pdf), JDXA provides fast, flexible, lightweight, and easy-to-use ORM functionality.

JDXA bridges the gap between the worlds of objects and relations with a simple and flexible solution. Some of the prominent features of JDXA include:

- Mapping between an object model and a relational model (ORM Specification) is defined textually using a simple grammar that requires minimal specification. JDXA User Manual has the full grammar specification.
- Full flexibility in domain object modeling – one-to-one, one-to-many, and many-to-many relationships as well as class-hierarchies supported.
- A small yet powerful set of APIs that application developers can effectively use to integrate their object-oriented applications with relational databases.
- POJO (Plain Old Java Objects) friendly non-intrusive programming model, which does not require you to change your Java classes in any way:
  - No need to subclass your domain classes from any base class
  - No need to clutter your source code with annotations
  - No need for DAO classes
  - No source code generation
  - No pre-processing or post-processing of your code
- Automatic database schema creation based on the ORM specification.
- A highly optimized metadata-driven object-relational mapping engine that is lightweight, dynamic, and flexible.

There are just 3 simple steps to use JDXA:

1. Define domain object model (Java classes)
2. Define a declarative object-relational mapping specification textually
3. Develop applications using intuitive and powerful JDXA APIs

On top of the core ORM functionality of JDX, we have created many Android platform-specific utility classes to facilitate the easy and speedy development of mobile apps that need on-device access to SQLite relational database.

### [**Download JDXA ORM SDK**](http://softwaretree.com/v1/products/jdxa/download-jdxa.php)

The JDXA SDK ships with plenty of documentation including the JDXA User Manual and Javadocs. The SDK also ships with many readymade sample application projects exemplifying the typicla ORM-related structure of Android applications, ORM specification files, usage of Android-specific utility classes, and JDXA API calls.

The following discussions provide some details about installing and using JDXA ORM SDK for Android app development.

## **Installation**

Please unzip the JDXA ORM SDK distribution jar file in a new folder (for example, JDXForAndroid). You will see the following directory structure:

```
JDXForAndroid\
    JDXForAndroid_Introduction.pdf
    libs\
    docs\
    examples\
    SQLDroidLocal\
```
The libs directory has the jar file JDXAndroid-nn.n.jar containing compiled code of JDXA ORM and associated utilities where nn.n denotes the version of JDXA. For example, JDXAndroid-01.5.jar denotes JDXA version 01.5 and JDXAndroid-02.0.jar denotes JDXA version 02.0.

The docs directory contains the Javadocs on JDXA ORM and associated utilities from an application developer's point of view. This directory also contains the JDXA user manual.

The examples directory contains many sample Eclipse projects that exemplify the usage of JDXA ORM in Android applications.

The SQLDroidLocal directory has the JDBC driver (SQLDroid) library (SQLDroid\bin\sqldroid.jar) based on the open-source code available at [GitHub](https://github.com/SQLDroid/SQLDroid). This library is provided under the terms of [The MIT License (MIT)](http://opensource.org/licenses/MIT).

## **Using JDXA in your Android project**

To use JDXA in your Android project, just add the libraries JDXAndroid-nn.n.jar (shipped in the libs directory of the SDK) and sqldroid.jar (shipped in the SQLDroidLocal\SQLDroid\bin directory of the SDK) as external jars. For example,

-  **In Eclipse:**

1. Put the above two jar files (JDXAndroid-nn.n.jar and sqldroid.jar) in your project's Java Build Path ( **Properties**  **→**  **Java Build Path**  **→**  **Libraries**  **→**  **Add External Jars…** ).

-  **In Android Studio:**

1. Create, if not already present, a directory named libs under the app directory of your project. Add the above two jar files (JDXAndroid-nn.n.jar and sqldroid.jar) in that libs directory.
2. Within Android Studio, under the **Project** view, locate the JDXAndroid-nn.n.jar file in the libs directory and **right click** on it and then **click** on the menu item **Add as Library…**. You will see the dependency on this jar file added to your build.gradle file as:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_compile files('libs/JDXAndroid-nn.n.jar')_

3. Repeat the above step (2) for the sqldroid.jar file.

After setting up these jar files as external libraries, you can start using JDXA APIs in your Android project.

## **Simple Example Code**

Just to give you a flavor of coding simplicity achieved by using JDXA ORM, this section provides representative example code of an Android application accessing an on-device SQLite database. Any UI and exception handling code is not provided in these code excerpts. There are just 3 simple steps to use JDX:

**1: Define domain object model (Java classes)**

JDXA does not impose any restriction in terms of how a class should be defined. Instances of any POJO (Plain Old Java Object) can be persisted using JDXA. Here is an example of a class definition (com.mycompany.model.Employee)
```java
package com.mycompany.model;
import java.util.Date;

public class Employee {
    private int id;
    private String name;
    private Date DOB;
    private boolean exempt;
    private float**compensation;

    // A no-arg constructor needed for JDX
    public Employee() {
    }
    // Define other constructors, and setters/getters
}
```
_Domain model class file Employee.java_


**2. Define a declarative object-relational mapping specification textually**

Here is an example of the mapping specification contained in a text file (say example.jdx), which should be located in the res/raw directory of the Android project.

- The name of the database is example.db, which is created/located in the default database directory for the package com.mycompany at /data/data/com.mycompany/databases/.
- A CLASS specification mentioning the fully qualified class name encapsulates all the ORM information for that class.  A primary key specification is mandatory for each class.
- The default column name and type for each class attribute (field) is automatically deduced by JDXA.  The SQLMAP specification below is to change the column name of the table from the default 'compensation' to 'salary'.  If the default column name were to be acceptable (as in case of other attributes), the SQLMAP specification below would not be needed.
- By default, JDXA treats all those attributes, which are neither static, nor transient, nor volatile to be persistent. However, by using an IGNORE statement (not shown here) in the mapping specification, one can specify a list of 'regular' attributes that JDXA should ignore for persistence.

```java
JDX_DATABASE JDX:jdbc:sqldroid:/data/data/com.mycompany/databases/example.db;…
;
CLASS com.mycompany.model.Employee TABLE Employee
   PRIMARY_KEY id
   SQLMAP FOR compensation COLUMN_NAME Salary
;
```
_Mapping file example.jdx_


**3: Develop applications using intuitive and powerful JDXA APIs**

The following code gives an example of how the core and utility JDXA classes described earlier can be used to initialize an on-device SQLite database and the ORM system, and to interact with the database in an intuitive object-oriented way that does not involve writing any complex SQL code. The AppSpecificJDXSetup utility class (code not shown) is used to easily initialize the database (example.db) and the ORM system based on the mapping specification (example.jdx).

```java
public class JDXAndroidExampleActivity extends Activity {
    JDXSetupjdxSetup = null;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout._main_);
        try {
          AppSpecificJDXSetup.initialize();
          jdxSetup = AppSpecificJDXSetup._getInstance_(this);
          useJDXORM(jdxSetup);
       } catch (Exception ex) { // Handle exception
           cleanup();
           return;
       }
    }

    private void cleanup() {
       AppSpecificJDXSetup._cleanup_();
       jdxSetup = null;
    }

    private void useJDXORM(JDXSetup jdxSetup) throws Exception {
        JDXHelper jdxHelper = new JDXHelper(jdxSetup);
        String employeeClassName = Employee.class.getName();
       
        // First delete all existing employees from the database.
        jdxHelper.delete2(employeeClassName, null);
       
        // Create and save a new employee Mark
        Employee emp = new Employee(1, "Mark", …);
        jdxHelper.insert(emp, false);

        // Create and save a new employee Bill
        emp = new Employee(2, "Bill", …);
        jdxHelper.insert(emp, false);

        // Retrieve all the employees
        List queryResults = jdxHelper.getObjects(employeeClassName, null);

        // Retrieve employee Bill (id=2)
        emp = (Employee) jdxHelper.getObjectById(employeeClassName, "id=2", ...)

        // Change and update attributes of Bill
        emp.setExempt(true);
        jdxHelper.update(emp, false);

        return;
    }
}
```
_Android activity programming file JDXAndroidExampleActivity.java_

## **JDXA Utility Classes**

_JDXA_ is shipped with many convenient utility classes (in the package com.softwaretree.jdxandroid) to simplify development of Android apps.  These utilities interact with the Android platform-specific artifacts and with the JDXA ORM framework to present a convenient application development interface.  An application can use those utilities alone or use those utilities along with JDX Core as per its needs.  Here are brief descriptions of some of those utility classes:

**JDXSetup**

This class helps set up and interact with the JDXA ORM system on the Android platform.

**BaseAppSpecificJDXSetup**

This is a utility class whose subclass can be used to initialize the underlying database and to create, share, and cleanup a singleton JDXSetup object in an Android application across multiple activities.

**JDXHelper**

This class provides a useful facade (wrapper) for many of the JDXA ORM methods.  You may use this class as is, subclass it to create a more exhaustive wrapper, or use the methods in this class as templates for your own code.  Here are some of the methods in the JDXHelper class:

```java
    public List getObjects(String className, String predicate)

    public int getObjectCount(String className, String attribName, String predicate)

    public void insert(Object object, boolean deep)

    public void update(Object object, boolean deep)

    public void delete(Object object, boolean deep)

    public long getNextSeq(String seqName, long increment)

    public void setJDXLogging(String jdxLogFileName)
```

**SimpleObjectListAdapter**

This is an adapter utility class to retrieve and view objects of a given class in ListActivity.  Objects are retrieved using the JDXA ORM system. A subclass would implement the abstract methods to use the retrieved objects in appropriate views.

**SimpleStreamingObjectsListAdapter**

This is a utility object list adapter to retrieve and view a polymorphic stream of objects of a given class in ListActivity.  Objects are fetched in chunks (stream) using the JDXA ORM. A subclass would implement the abstract methods to use the retrieved objects in appropriate views.

**JDXSeqUtil**

This is a utility class to easily mange persistently unique Named Sequence values.

**JDX\_JSONObject**

This class serves as a base class for persisting JSONObject instances of a domain specific class which should be defined as a subclass of this class.  The object relational mapping for that domain specific class is defined using VIRTUAL\_ATTRIB specifications for each persistent attribute of the corresponding JSON object.

**Utils**

This class has some nifty utility methods that can be used by JDXA apps.

## **Code Snippets**

Here are links to some programming snippets to illustrate JDXA ORM mapping specifications and APIs for sample object models (class definitions). Each sample is named to signify a typical modeling or usage pattern.

In the interest of keeping our description short and to the point, we may not show different constructors and accessor (setter/getter) methods in a class definition. Similarly, although a mapping specification (provided in a text file) may contain mapping information for multiple classes used in a particular application, we might just show and concentrate on the mapping information for only one or a few classes. Also, we may not show any business logic or user interface (UI) related code.

[**Simple Mapping 1**](http://www.softwaretree.com/2015/products/jdxa/simple_mapping1.html)

This snippet shows how to define default mapping for a POJO (Plain Old Java Object) class. It also shows JDXA ORM programming code of how instances of such a class can easily be inserted, deleted, updated, and retrieved in an intuitive object oriented way.

We use a simple Employee class for illustration.

[**Simple Mapping 2**](http://www.softwaretree.com/2015/products/jdxa/simple_mapping2.html)

Similar to Simple Mapping 1, this snippet additionally shows how to specify a non-default table name, how to specify the table column names to be different than the class attribute names, and how to ignore some attributes for persistence.

[**Mapping One-To-One Relationships**](http://www.softwaretree.com/2015/products/jdxa/mapping_one_2_one.html)

This snippet shows how to define one-to-one relationships. It also shows example of deep and shallow operations involving related objects.

In the example object mode, an employee has an address and works in a department.

[**Mapping One-To-Many Relationships**](http://www.softwaretree.com/2015/products/jdxa/mapping_one_2_many.html)

This snippet shows how to define collection classes and one-to-many relationships. It also shows example of deep and shallow operations involving related objects.

In the example class model, a company has many departments.

[**Mapping Many-To-Many Relationships**](http://www.softwaretree.com/2015/products/jdxa/mapping_many_2_many.html)

This snippet shows how to define collection classes, join classes, and many-to-many relationships. An intermediate join class is used to hold the references for a many-to-many relationship. The snippet also shows examples of deep and shallow operations involving related objects. For a many-to-many relationship, the entities involved in the relationship are considered independent in the sense that insert, update, and delete operations don't go beyond the intermediate join class (table).

In the example object model, one user has many groups and one group has many users.

[**Mapping Auto-Increment Columns for Primary Key**](http://www.softwaretree.com/2015/products/jdxa/Mapping-Auto-Increment-Columns-for-Primary-Key.html)

This snippet shows how to define and use auto-increment columns for generating primary keys for objects of a class. JDXA does not set values of the corresponding columns through the INSERT and UPDATE statements used for storing or updating the instances of this class. However, the query operations do fetch the corresponding column values.

JDX also provides a named sequence generator facility that can alternatively be used to efficiently generate unique ids that you can assign to your objects before saving them in the database.

[**Mapping JSON Objects**](http://www.softwaretree.com/2015/products/jdxa/Mapping-JSON-Objects.html)

This snippet shows how to define classes and mapping specifications for persisting JSON objects in a database using JDXA ORM. It also shows JDXA ORM programming code of how JSON objects can easily be inserted, deleted, updated, and retrieved in an intuitive object oriented way.

Although this snippet shows how JDXA can help you with persistence of simple JSON objects, JDXA can also help you easily handle complex JSON objects having other nested JSON objects and arrays of JSON objects.



## **Sample Projects**

The JDXA ORM SDK ships with many sample projects of working Android applications, which use JDXA for data integration.  These projects provide examples of many different features of the JDXA ORM product including how to specify mappings for domain model classes having one-to-one, one-to-many, and many-to-many relationships, how to use sequence generators for unique ids, how to use object-oriented list view classes, how to automatically create the underlying database schema (tables and constraints) as per the mapping specification, and how to leverage various JDX APIs to easily interact with a relational database in an object-oriented way.

These projects are conveniently located under the examples directory.  You may have to adjust the build/library paths for these projects depending upon the location of your SDK installation.  Many of these applications produce some illustrative output in either the LogCat panel of the Eclipse IDE (Window → Show View → Other, Android → LogCat) or on the screen.

High-level descriptions of some sample projects follow. Please see the actual code and the documentation for more details.

Some of these sample projects are also hosted on GitHub. The names of such projects are linked to the appropriate GitHub URL.

All the sample projects are provided as Eclipse IDE projects. However, an X in the **Also Available as Android Studio Project** column means that the SDK also contains that sample project for Android Studio IDE (in the directory examples/ AndroidStudioJDXAProjects).

| **Sample Project** | **Brief Description** | **Also Available as Android Studio Project** |
| --- | --- | --- |
| [_JDXAndroidSimpleExample_](https://github.com/SoftwareTree/JDXAndroidSimpleExample) | Demonstrates how JDXA ORM and associated utilities can be used to easily develop an Android app that exchanges data of domain model objects with an SQLite database. | X |
| _JDXAndroidSimpleExample2_ | Demonstrates how a JDXHelper object can be used to interact with relational data using simpler methods. | X |
| [_JDXAndroidRelationshipsExample_](https://github.com/SoftwareTree/JDXAndroidRelationshipsExample) | Demonstrates using JDXA for an object model with one-to-one and one-to-many relationships. | X |
| _JDXAndroidRelationships2Example_ | Demonstrates using JDXA for persisting an object model such that the attribute values of the related objects are stored in the same table (INLINE or EMBEDDED) where the attribute values of the parent object are stored.  | X |
| [_JDXAndroidAutoIncrementExample_](https://github.com/SoftwareTree/JDXAndroidAutoIncrementExample) | Demonstrates JDXA facilitating the use of an autoincrement column for a primary key attribute of an object. | X |
| _JDXAndroidSimpleLoginExample_ | Demonstrates how to easily develop a user login/signup subsystem for an Android app. | X  |
| _JDXAndroidClassHierarchyExample_ | Demonstrates using JDXA for an object model with class hierarchies. Person is the superclass with BaseEmployee and Intern has its subclasses. BaseEmployee has further two subclasses - PermEmployee and TempEmployee. | X |
| [_JDXAndroidImagesExample_](https://github.com/SoftwareTree/JDXAndroidImagesExample) | Demonstrates using JDX with image data. | X |
| _JDXAndroidListExample1_ | Demonstrates using a JDXA provided ListAdapter class to query a list of objects from the database and displaying them. | X |
| _JDXAndroidListExample2_ | Demonstrates using a JDX provided ListAdapter class to query a filtered list of objects from the database and displaying them. | X  |
| _JDXAndroidStreamingExample_ | Demonstrates use of streaming queries to retrieve a list of objects in separate chunks as needed. | X  |
| _JDXAndroidStreamingListExample_ | Demonstrates using a JDXA provided streaming ListAdapter class to query a list of objects from the database and displaying them iteratively on demand by fetching only a few objects from the database at a time. |   |
| _JDXAndroidSequencesExample_ | Demonstrates defining named sequences for generating persistently unique sequence numbers and using the JDXSeqUtility class to easily and efficiently create unique keys at runtime. |  X  |
| [_JDXAndroidManyToManyExample_](https://github.com/SoftwareTree/JDXAndroidManyToManyExample) | Demonstrates using JDXA for an object model with many-to-many relationships.   | X  |
| _JDXAndroidJSONExample_ | Demonstrates using JDXA for persistence of JSON objects. | X  |
| _JDXAndroidListWithHolderPatternExample_ | Demonstrates using JDXA for retrieving a list of objects from databases, employing a holder pattern to cache and use the references to widgets for displaying an  object in a list view, and persisting an updated object back into the database. | X  |
| _JDXAndroidPrePostMethodsExample_ | Demonstrates using JDXPreInsert, JDXPresUpdate, and JDXPostQuery callback methods in a domain model class to automatically massage (for example, encode, decode, compress, and decompress) instance data (perhaps for security and saving disk space reasons) before saving it to the database and after retrieving it from the database. | X  |
| _JDXAndroidAsyncQueryExample_ | Demonstrates using JDXA for retrieving object(s) asynchronously in a background thread and then showing them in a UI (main) thread. | X  |
| _JDXAndroidCachingExample_ | Demonstrates how, with JDXA, in-memory caches may be established for particular classes such that the database need not be queried for previously cached objects. | X  |
| _JDXAndroidDBUpgradeV1Example JDXAndroidDBUpgradeV2Example_ | Demonstrate a strategy to facilitate easy migration of the existing data produced from an old version of an app for use in the new version of the app. In this strategy, any needed data from the old (existing) database is migrated to a new database, which is then used by the new version of the app. The old database remains unchanged and may be discarded or may be used as a backup resource. | X  |