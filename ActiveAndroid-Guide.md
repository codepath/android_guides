## Overview

Using the ActiveAndroid ORM makes managing client-side models extremely easy in simple cases. For more advanced or custom cases, you can use [SQLiteOpenHelper](http://www.androidhive.info/2011/11/android-sqlite-database-tutorial/) to manage the database communication directly. But for simple model mapping from JSON, ActiveAndroid keeps things simple. 

Using ActiveAndroid is as simple as the following steps:

 * [Install ActiveAndroid and perform initial setup](https://github.com/pardom/ActiveAndroid/wiki/Getting-started)
 * [Define your application models](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model)
 * [Persist data using save](https://github.com/pardom/ActiveAndroid/wiki/Saving-to-the-database)
 * [Query the local database](https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database)
 * [Perform data schema migrations](https://github.com/pardom/ActiveAndroid/wiki/Schema-migrations)

## Common Questions

> How does ActiveAndroid handle duplicate IDs?  For example, I want to make sure no duplicate twitter IDs are inserted.  Is there a way to specify a column is primary key in the model?

The solution is to mark the column recording the unique id of an object as a unique column acting as your pseudo primary key. As [explained here](https://github.com/pardom/ActiveAndroid/issues/22), the annotation is:

```java
@Table(name = "items")
public class SampleModel extends Model {
    // Ensure the remoteId must be unique. If a duplicate tries to save, replace the 
    @Column(name = "remote_id", unique = true, onUniqueConflict = Column.ConflictAction.REPLACE)
    private int remoteId;

    // ... set the remote id based on the json response
}
```

Make sure to **uninstall the app** afterward on the emulator to ensure the schema changes take effect.

> I read somewhere that ActiveAndroid automatically creates another auto-increment ID column, is this true?  What field names should I avoid using?

This is true, see https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model for more details. The column is called ["mId"](https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/Model.java#L40)

> How do you specify the data type (int, text)?  Does AA automatically know what the column type should be?

Inferred automatically from the type of the field

> How do you represent a 1-1 relationship?  

Check out https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model#relationships if you haven't yet. Basically there are many ways to manage this. You can declare a type "User" and then that field will be associated with a foreign key representing the user: https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/Model.java#L135. You manage this process more manually by simply constructing and assigning the user object and then calling save on the parent. You do need to call save separately in most cases.

> How do I delete all the records from a table?

You can delete individual records using the `.delete` method and you can delete all records matching a particular condition with:

```java
new Delete().from(Item.class).execute(); // all records
// or based on a conditional clause
new Delete().from(Item.class).where("Id = ?", 1).execute();
```

This allows for bulk deletes.

> Is it possible to do joins with ActiveAndroid? 

Joins are done using the query language AA provides: https://github.com/pardom/ActiveAndroid/blob/master/src/com/activeandroid/query/From.java read more here: https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database

> What are the best practices when interacting with the sqlite in Android, is ORM/DAO the way to go?

I've seen people use both SQLiteOpenHelper and several different ORMs. It's common to use the SQLiteOpenHelper in cases where an ORM breaks down or isn't necessary. Since Models are typically formed anyways though and persistence on Android in many cases can map very closely to objects, ORMs like AA can be helpful especially for simple mappings.

## References

* [ActiveAndroid Website](http://www.activeandroid.com/)
* [AA Getting Started](https://github.com/pardom/ActiveAndroid/wiki/Getting-started)
* [AA Models](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model)
* [AA Saving](https://github.com/pardom/ActiveAndroid/wiki/Saving-to-the-database)
* [AA Querying](https://github.com/pardom/ActiveAndroid/wiki/Querying-the-database)
* [AA Migrations](https://github.com/pardom/ActiveAndroid/wiki/Schema-migrations)
* [AA Pre-populated](https://github.com/pardom/ActiveAndroid/wiki/Pre-populated-databases)