### Overview

One of the issues with existing SQL object relational mapping (ORM) libraries is that they rely on Java reflection to define database models, table schemas, and column relationships.   [DBFlow](https://github.com/Raizlabs/DBFlow) is one of the few that relies strictly on annotation processing to generate Java code based on the [[SQLiteOpenHelper|Local-Databases-with-SQLiteOpenHelper]] framework that avoids this issue.   This approach results in increasing run-time performance while also saving you from having to write a lot boilerplate code normally needed to declare your tables, manage their schema changes, and perform queries.

### Setup

The section below describes how to setup using DBFlow v3, which is currently in beta but still very much ready for production use.  If you are upgrading from an older version of DBFlow, read this [migration guide](https://github.com/Raizlabs/DBFlow/blob/master/usage/Migration3Guide.md).  One of the major changes is the library used to generate Java code from annotation now relies on [JavaPoet](https://github.com/square/javapoet).  The generated database and table classes now use '_' instead of '$' as the separator, which may require small adjustments to your code when upgrading.

#### Gradle configuration

The first step is to add the android-apt plugin to your root `build.gradle` file:

```gradle
buildscript {
    repositories {
      // required for this library, don't use mavenCentral()
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
```

Because DBFlow3 is still not officially released, you need also add https://jitpack.io to your `allprojects` -> `repositories` dependency list:

```java
allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
}
```

Next, within your `app/build.gradle, apply the `android-apt` plugin and add DBFlow to your dependency list.  We create a separate variable to store the version number to make it easier to change later:

```gradle
apply plugin: 'com.neenbedankt.android-apt'

def dbflow_version = "3.1.1"

dependencies {
    apt "com.github.Raizlabs.DBFlow:dbflow-processor:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow-core:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow:${dbflow_version}"

    // sql-cipher database encryption (optional)
    // compile "com.github.Raizlabs.DBFlow:dbflow-sqlcipher:${dbflow_version}"
  }
```

**Note:** Make sure to remove any previous references of DBFlow if you are upgrading.  The annotation processor has significantly changed for older versions.  If you `java.lang.NoSuchMethodError: com.raizlabs.android.dbflow.annotation.Table.tableName()Ljava/lang/String;`, then you are likely to still have included the old annotation processor in your Gradle configuration.

#### Instantiating DBFlow

Next, we need to instantiate `DBFlow` in our main application.  If you do not have an `Application` object, create one:

```java

public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        FlowManager.init(new FlowConfig.Builder(this).build());
    }
}
```

Modify your `AndroidManifest.xml` file to call this Application object:

```xml
<application
        android:name=".MyApplication"
</application>
```

If you skip this step, you may see `com.raizlabs.android.dbflow.structure.InvalidDBConfiguration: Model object: XXXX is not registered with a Database. Did you forget an annotation?`.

#### Creating the database

Create a `MyDatabase.java` file and annotate your class with the `@Database` decorator to declare your database.  It should contain both the name to be used for creating the table, as well as the version number.  **Note**: if you decide to change the schema for any tables you create later, you will need to bump the version number.  The version number should always be incremented (and never downgraded) to avoid conflicts with older database versions.
 
```java
@Database(name = MyDatabase.NAME, version = MyDatabase.VERSION)
public class MyDatabase {

    public static final String NAME = "MyDataBase";

    public static final int VERSION = 1;
}
```

#### Defining your Tables

The Java objects that need to be declared as models need to extend from `BaseModel`.  In addition, you should annotate the class with the database name too.   Here we show how to create an `Organization` and `User` table:

```java
import com.raizlabs.android.dbflow.structure.BaseModel;

@Table(database = MyDatabase.class)
public class Organization extends BaseModel {

  @Column
  @PrimaryKey
  int id;

  @Column
  String name;
}
```

We can define a `ForeignKey` relation easily.  The `saveForeignKeyModel` denotes whether to update the foreign key if the entry is also updated.  In this case, we disable this functionality:

```java
@Table(database = MyDatabase.class)
public class User extends BaseModel {

    @Column
    @PrimaryKey
    int id;

    @Column
    String name;

    @Column
    @ForeignKey(saveForeignKeyModel = false)
    Organization organization;

    public void setOrganization(Organization organization) {
        this.organization = organization;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

#### Using with the Parceler library

If you are using DBFlow with the [[Parceler|Using Parceler]] library, make sure to annotate the class with the `@Parcel(analyze={}` decorator.  Otherwise, the Parceler library will try to serialize the fields that are associated with the `BaseModel` class and trigger `Error:Parceler: Unable to find read/write generator for type` errors.  To avoid this issue, specify to Parceler exactly which class in the inheritance chain should be examined (see this [disucssion](https://github.com/johncarl81/parceler/issues/73#issuecomment-167131080) for more details):

```java
@Table(database = MyDatabase.class)
@Parcel(analyze={User.class})   // add Parceler annotation here
public class User extends BaseModel {
}
```

### Basic CRUD operations

Basic creation, read, update, and delete (CRUD) statements are fairly straightforward to do.  DBFlow generates a Table class for each your annotated models (i.e. User_Table, Organization_Table), and each field is defined as a `Property` object and ensures type-safety when evaluating it against in a SELECT statement or a raw value.

In order to perform queries, make sure to compile your code so that the tables and column names can be generated.   See [this section](https://github.com/Raizlabs/DBFlow/blob/master/usage2/SQLiteWrapperLanguage.md) for more details on the queries that can be performed (i.e. sum, count, AND/OR, and joins).

#### Creating rows

We can simply call `.save()` on the annotated class to save the row into the table:

```java
// Create organization
Organization organization = new Organization();
organization.setId(1);
organization.setName("CodePath");
organization.save();

// Create user
User user = new User();
user.setName("John Doe");
user.setOrganization(organization);
user.save();
```

#### Reading rows

```java
List<Organization> organizationList = new Select().from(Organization.class).queryList();
List<User> users = new Select().from(User.class).where(Organization_Table.name.is("CodePath")).queryList();
```

#### Updating rows

Calling `save()` on the object will automatically update if the record has already been saved and there is a primary key that matches.

#### Deleting rows

We can simply call `.delete()` on the respective object:

```java
user.delete();
```

### Database Transactions

See [this guide](https://github.com/Raizlabs/DBFlow/blob/master/usage/StoringData.md) about how to perform transactions.  You can batch save a list of `User` objects by using the `ProcessModelTransaction` class:

```java

ArrayList<User> users = new ArrayList<>();

// fetch users from the network

// save rows
FlowManager.getDatabase(AppDatabase.class)
          .beginTransactionAsync(new ProcessModelTransaction.Builder<>(
          new ProcessModelTransaction.ProcessModel<User>() {
              @Override
              public void processModel(User user) {
                   // do work here -- i.e. user.delete() or user.update()
                   user.save(); 
              }
          }).addAll(users).build())  // add elements (can also handle multiple)
          .error(new Transaction.Error() {
              @Override
              public void onError(Transaction transaction, Throwable error) {

              }
          })
          .success(new Transaction.Success() {
              @Override
              public void onSuccess(Transaction transaction) {

              }
          }).build().execute();

```
### Exposing Content Providers

One of the side benefits of using DBFlow is that you can expose tables easily as Android [[Content Providers|Creating-Content-Providers]], which enables other apps to query this data.    

The first step is to declare these Content Providers in the same place where your database is declared to help centralize all the declarations.  We also need to expose a URL for other apps to query, which will be declared as `content://com.codepath.myappname.provider` in this example.

```java
@ContentProvider(authority = MyDatabase.AUTHORITY,
        database = MyDatabase.class,
        baseContentUri = MyDatabase.BASE_CONTENT_URI)
@Database(name = MyDatabase.NAME, version = MyDatabase.VERSION)
public class MyDatabase {

    public static final String NAME = "MyDatabase";

    public static final int VERSION = 1;

    public static final String AUTHORITY = "com.codepath.myappname.provider";

    public static final String BASE_CONTENT_URI = "content://";

    private static Uri buildUri(String... paths) {
        Uri.Builder builder = Uri.parse(AppDatabase.BASE_CONTENT_URI + AppDatabase.AUTHORITY).buildUpon();
        for (String path : paths) {
            builder.appendPath(path);
        }
        return builder.build();
    }
}
```

Next, within this same class, we will declare a `User` endpoint (i.e. content://com.codepath.myappname.provider/User) that can be declared:

```java
public class MyDatabase {

    // ...

    // Declare endpoints here
    @TableEndpoint(name = UserProviderModel.ENDPOINT, contentProvider = MyDatabase.class)
    public static class UserProviderModel {

        public static final String ENDPOINT = "User";

        @ContentUri(path = IdentityProviderModel.ENDPOINT,
                type = ContentUri.ContentType.VND_MULTIPLE + ENDPOINT)
        public static final Uri CONTENT_URI = buildUri(ENDPOINT);
    }
}
```

#### Adding to Manifest file

The final step is for the Content Provider to be exposed.  If you wish for other apps to be able to view this data, set `android:exported` to be true.  Otherwise, if you only wish the existing application to query this content provider, set the value to be false.  Note that DBFlow3 uses underscore (`_`) instead of the dollar sign (`$`) as the separator:

```xml
<provider
            android:authorities="com.codepath.myapp.provider"
            android:exported="true|false"
            android:name=".provider.TestContentProvider_Provider"/>
```

## Common Questions

> Question: How do I inspect the SQLite data stored on the device?

In order to inspect the persisted data, we need to [[use adb to query or download the data|Local-Databases-with-SQLiteOpenHelper#sqlite-database-debugging]].  You can also take a look at using the [[Stetho|Troubleshooting-Common-Issues#database-inspection]] library, which provides a way to use Chrome to inspect the local data.

## Common Questions

> Question: How do I inspect the SQLite data stored on the device?

In order to inspect the persisted data, we need to [[use adb to query or download the data|Local-Databases-with-SQLiteOpenHelper#sqlite-database-debugging]].  You can also take a look at using the [[Stetho|Troubleshooting-Common-Issues#database-inspection]] library, which provides a way to use Chrome to inspect the local data.

> Question: How does DBFlow handle duplicate IDs?  For example, I want to make sure no duplicate twitter IDs are inserted.  Is there a way to specify a column is the primary key in the model?

Simply annotate the post ID column as the `@PrimaryKey` annotation:

```java
@Table(database = MyDatabase.class)
public class SampleModel extends BaseModel {

    @PrimaryKey
    @Column
    private int remoteId;

    // ... set the remote id based on the json response
}
```

Make sure to **uninstall the app** afterward on the emulator to ensure the schema changes take effect. Note that you may need to manually ensure that you don't attempt to re-create existing objects by verifying they are not already in the database **as shown below**.

> Question: How do you specify the data type (int, text)?  Does DBFlow automatically know what the column type should be?

The type is inferred automatically from the type of the field.

> Question: How do I store dates into DBFlow?

DBFlow supports serializing Date fields automatically. It is stored internally as a timestamp (INTEGER) in milliseconds.

```java
@Column
private Date timestamp; 
```

and the date will be serialized to SQLite. You can parse strings into a Date object using [SimpleDateFormat](http://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html):

```java
public void setDateFromString(String date) {
    SimpleDateFormat sf = new SimpleDateFormat("EEE MMM dd HH:mm:ss ZZZZZ yyyy");
    sf.setLenient(true);
    this.timestamp = sf.parse(date);
}
```

```java
public static List<Model> findRecent(Date newerThan) {
    return new Select().from(Model.class).where("timestamp > ?", newerThan.getTimeInMillis()).execute();
}
```

> Question: How do you represent a 1-1 relationship?  

Check out the [relationships section](https://github.com/Raizlabs/DBFlow/blob/master/usage2/Relationships.md) if you haven't yet.  You will need to annotate the field with the `@ForeignKey` and `@Column` annotation:

```java
public class User extends BaseModel {
  public String email;

  @Column
  @ForeignKey
  public Address address;
}
```

You can manage this process by simply constructing and assigning the user object to a field of the parent object and then calling save on the parent. 

```java
User u = new User("bob@foo.com");
u.address = new Address("135 Hesby St, Los Angeles, CA");
u.address.save();
u.save();
```

You should make sure to **call save separately on the associated object** in most cases.

> Question: How do I delete all the records from a table?

See [this section](https://github.com/Raizlabs/DBFlow/blob/master/usage2/SQLiteWrapperLanguage.md#delete) of DBFlow.  You can delete individual records using the `.delete` method and you can delete all records matching a particular condition with:

```java
// Delete a whole table
Delete.table(MyTable.class);

// Delete multiple instantly
Delete.tables(MyTable1.class, MyTable2.class);

// Delete using query
SQLite.delete(MyTable.class)
  .where(DeviceObject_Table.carrier.is("T-MOBILE"))
    .and(DeviceObject_Table.device.is("Samsung-Galaxy-S5"))
  .async()
  .execute();
```

This allows for bulk deletes.

> Question: Is it possible to do joins with DBFlow? 

Joins are done using the query language DBFlow provides in the [From class](https://github.com/Raizlabs/DBFlow/blob/master/usage2/SQLiteWrapperLanguage.md#joins). You can read more here on the [SQLiteWrapperLanguage docs](https://github.com/Raizlabs/DBFlow/blob/master/usage2/SQLiteWrapperLanguage.md)

> Question: What are the best practices when interacting with the sqlite in Android, is ORM/DAO the way to go?

Developers use both [[SQLiteOpenHelper|Local-Databases-with-SQLiteOpenHelper]] and [[several different ORMs|Persisting-Data-to-the-Device#object-relational-mappers]]. It's common to use the SQLiteOpenHelper in cases where an ORM breaks down or isn't necessary. Since Models are typically formed anyways though and persistence on Android in many cases can map very closely to objects, ORMs like ActiveAndroid can be helpful especially for simple database mappings.

## Troubleshooting

Because DBFlow requires on annotation processing, sometimes you may need to click `Build` -> `New Project` to rebuild the source code generated.  

### ProGuard issues

If you are using DBFlow with [[ProGuard|Configuring ProGuard]], and see `Table is not registered with a Database. Did you forget the @Table annotation?`, make sure to include this line in your ProGuard configuration:

```
-keep class * extends com.raizlabs.android.dbflow.config.DatabaseHolder { *; }
```

You can go into your `app/build/intermediate/classes/com/raizlabs/android/dbflow/config` and look for the `GeneratedDatabaseHolder.class` to understand what code is generated.  

### Database schema

You can also download and inspect the local database using ADB:

```bash
adb pull /data/data/com.codepath.yourappname/databases/MyDatabase*.db
sqlite3 MyDatabase.db 
```

Type `.schema` to see how the table definitions are created:

```
sqlite> .schema
CREATE TABLE android_metadata (locale TEXT);
CREATE TABLE `Organization`(`id` INTEGER,`name` TEXT, PRIMARY KEY(`id`));
CREATE TABLE `User`(`id` INTEGER,`name` TEXT,`organization_id` INTEGER, PRIMARY KEY(`id`), FOREIGN KEY(`organization_id`) REFERENCES `Organization`(`id`) ON UPDATE NO ACTION ON DELETE NO ACTION);
```

### Using with Gson Library

If you intend to use DBFlow with the [[Gson|Leveraging-the-Gson-Library]] library, you may get `StackOverflowError` exceptions when trying to use Java objects that extend from `BaseModel`.  In order to avoid these issues, you need to exclude the `ModelAdapter` class, which is a field included with [BaseModel](https://github.com/Raizlabs/DBFlow/blob/master/dbflow/src/main/java/com/raizlabs/android/dbflow/structure/BaseModel.java#L45):

```java

public class DBFlowExclusionStrategy implements ExclusionStrategy {

    // Otherwise, Gson will go through base classes of DBFlow models
    // and hang forever.
    @Override
    public boolean shouldSkipField(FieldAttributes f) {
        return f.getDeclaredClass().equals(ModelAdapter.class);
    }

    @Override
    public boolean shouldSkipClass(Class<?> clazz) {
        return false;
    }
}
```

You then need to create a custom Gson builder to exclude this class:
 
```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setExclusionStrategies(new ExclusionStrategy[]{new DBFlowExclusionStrategy()});
```

See [this issue](https://github.com/Raizlabs/DBFlow/issues/121) for more information.

### References

* <https://github.com/Raizlabs/DBFlow#usage-docs>
* <https://github.com/Raizlabs/DBFlow/blob/master/usage2/SQLiteWrapperLanguage.md>