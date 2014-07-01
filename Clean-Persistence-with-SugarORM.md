[SugarORM](https://github.com/satyan/sugar) is a database persistence library that provides a simple and concise way to integrate your application models into SQLite. In contrast to [ActiveAndroid](https://github.com/pardom/ActiveAndroid), which is mature, powerful, and flexible, SugarORM is:

* Less verbose
* Quicker to set up
* More hands-free

## Install

SugarORM has no dependencies, so installation is as simple as downloading the `.jar` file and putting it in your `libs` folder. Maven central hosting is currently being [planned](https://github.com/satyan/sugar/issues/91), and this page will update with additional information whenever SugarORM becomes available on Maven.

The current stable release is [v1.2](https://github.com/satyan/sugar/blob/master/dist/sugar-1.2.jar?raw=true), but the beta release [v1.3_beta](https://github.com/satyan/sugar/blob/master/dist/sugar-1.3_beta.jar?raw=true) is highly recommended. Once you have the `.jar` file in your `libs` folder, finishing the installation requires just setting the `android:name` attribute of the `application` tag in your `AndroidManifest.xml`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk ... />
    <uses-permission ... />

    <application
        ...
        android:name="com.orm.SugarApp" >  <!--Use this attribute verbatim-->
        <meta-data android:name="DATABASE" android:value="example.db" />
        <meta-data android:name="VERSION" android:value="1" />
        <meta-data android:name="QUERY_LOG" android:value="true" />
        <meta-data android:name="DOMAIN_PACKAGE_NAME" android:value="com.example" />
        <activity ... />
        <activity ... />
    </application>
</manifest>
```

There are four additional parameters you can set. `DATABASE` is the name of the database file that will be created by SugarORM. `VERSION` is the version of the database schema. This is used for schema migrations, which are described in more detail below. `QUERY_LOG` can be set to `"true"` or `"false"` and determines whether to log debug messages for the queries made to the underlying SQLite database. Finally, `DOMAIN_PACKAGE_NAME` narrows down the packages that SugarORM scans for Classes to persist.

## Integrate

#### v1.2

Persisting your models is dead simple. Just extend `SugarRecord<YourModel>`, and change your constructors to take in a `Context` object as its first argument, and call `super(context)` right after the constructor declaration (examples adapted from [Designing Your Entities](http://satyan.github.io/sugar/creation.html)):

```java
public class Author extends SugarRecord<Author> {
    String fullName;
    int age;
    
    public Author(Context context) {
        super(context);
    }

    public Author(Context context, String fullName, int age) {
        super(context);
        this.fullName = fullName;
        this.age = age;
    }
}
```

And that's it! For camel cased field names, the underlying SQL database creates a column using underscores. For instance, `fullName` becomes `full_name` (this is important for querying).

Creating relationships between models also just *works*:

```java
public Book extends SugarRecord<Book> {
    Author author;
}
```

will create the appropriate column indicating a relationship between `Book` and `Author`.

To prevent a member field from being persisted in the SQLite database, you can ignore it with an annotation:

```java
public Author extends SugarRecord<Author> {
    String fullName;
    int age;

    @Ignore
    String password;
}
```

This will prevent `password` from having a column created for it in the underlying database.

#### v1.3 beta

While the above integration is simple, clean, and quick, integrating with v1.3 beta is strictly better. The only difference between the two versions in terms of their integration is that `Context` is no longer a required argument in v1.3 beta. A no argument constructor is the only remaining requirement. The above `Book` model would become:

```java
public Author extends SugarRecord<Author> {
    String fullName;
    int age;

    public Author() {
    }

    public Author(String fullName, int age) {
        this.fullName = fullName;
        this.age = age;
    }
}
```

This makes it even easier to drop in SugarORM into an existing project and have things just work right out of the box.

## Operations

The typical family of database operations has an interface in SugarORM. Most follow the format of `ModelName.operation(ModelName.class, arguments)`.

### Static methods

These operations can be performed on your Model class as a whole, requiring no instance of the Model itself.

#### Find by ID

```java
Author author = Author.findById(Author.class, 4L);
```

The `L` at the end of that number is necessary because `findById` expects a `Long` (an overloaded version that takes an `Integer` is in a [pull request](https://github.com/satyan/sugar/pull/113)).

#### Find all

```java
List<Author> allAuthors = Author.listAll(Author.class);
```

This query performs no filtering at all, so for tables with a lot of records, this operation may take a long time to complete.

#### Find with conditions

```java
List<Author> authors = Author.find(Author.class, "full_name = ?", "Nathan");
List<Book> books = Book.findWithQuery(Book.class, "Select * from Book limit ?", "4");
```

The first query embeds a `"where"` clause for convenience. The second query is a generic query that more closely proxies raw SQL. Both take a variable number of `String` arguments equal to the number of `?`'s in the query string.

#### Bulk delete

```java
Book.deleteAll(Book.class);
Author.deleteAll(Author.class, "age = ?", "31");
```

The first operation will, unsurprisingly, delete every `Book` record. The second operation only deletes `Author` records that have an age of `31`. Note that for inserting a value into the where clause, it's necessary to make it a `String`, even though the underlying column is an `int`.

#### Count (available in v1.3 beta only)

```java
long numberOfBooks = Book.count(Book.class, null, null);
long numberOfAuthors = Author.count(Author.class, "full_name = ?", "Timothy");
```

### Member methods

These operations are performed on a particular instance of the model.

#### Save

```java
Author author = new Author("J.K. Rowling", 48);
author.save();
```

This produces a record in the `Author` table with columns `full_name = "J.K. Rowling"` and `age = 48`.

#### Delete

```java
Author author = Author.findById(Author.class, 23L);
author.delete()
```

This removes the `Author` with `ID = 23` from the database. A common gotcha here is when there is no such `Author` with this `ID`, so a null pointer check here is often advisable.

### Query Builder

In addition to the above operations, SugarORM also comes with a query building interface. Using it allows you to separate the query step from the execution step. In this way, you can save a query and execute it multiple times, where each execution is done against the current database state.

## Migrations

## References

* <https://github.com/satyan/sugar>
* <https://github.com/pardom/ActiveAndroid> 
* <http://satyan.github.io/sugar/creation.html>