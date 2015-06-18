## Overview

Due to Android's memory management scheme, you will often find yourself needing to communicate with different components of your application, system components, or other applications installed on the phone.  `Parcelable` will help you pass data between these components. 

Android uses Binder to facilitate such communication in a highly optimized way.  The Binder communicates with Parcels, which is a message container.  The Binder marshals the Parcel to be sent, sends and receives it, and then unmarshals it on the other side to reconstruct a copy of the original Parcel.  

To allow for your class instances to be sent as a Parcel you must implement the `Parcelable` interface along with a static field called `CREATOR`, which itself requires a special constructor in your class.

### Defining a Parcelable Object

Here is a typical implementation:

```java
public class MyParcelable implements Parcelable {
    // You can include parcel data types
    private int mData;
    private String mName;
    
    // We can also include child Parcelable objects. Assume MySubParcel is such a Parcelable:
    private MySubParcelable mInfo;

    // This is where you write the values you want to save to the `Parcel`.  
    // The `Parcel` class has methods defined to help you save all of your values.  
    // Note that there are only methods defined for simple values, lists, and other Parcelable objects.  
    // You may need to make several classes Parcelable to send the data you want.
    @Override
    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(mData);
        out.writeString(mName);
        out.writeParcelable(mInfo, flags)
    }

    // Using the `in` variable, we can retrieve the values that 
    // we originally wrote into the `Parcel`.  This constructor is usually 
    // private so that only the `CREATOR` field can access.
    private MyParcelable(Parcel in) {
        mData = in.readInt();
        mName = in.readString();
        mInfo = in.readParcelable(MySubParcelable.class.getClassLoader());
    }

    public MyParcelable() {
        // Normal actions performed by class, since this is still a normal object!
    }

    // In the vast majority of cases you can simply return 0 for this.  
    // There are cases where you need to use the constant `CONTENTS_FILE_DESCRIPTOR`
    // But this is out of scope of this tutorial
    @Override
    public int describeContents() {
        return 0;
    }

    // After implementing the `Parcelable` interface, we need to create the 
    // `Parcelable.Creator<MyParcelable> CREATOR` constant for our class; 
    // Notice how it has our class specified as its type.  
    public static final Parcelable.Creator<MyParcelable> CREATOR
            = new Parcelable.Creator<MyParcelable>() {

        // This simply calls our new constructor (typically private) and 
        // passes along the unmarshalled `Parcel`, and then returns the new object!
        @Override
        public MyParcelable createFromParcel(Parcel in) {
            return new MyParcelable(in);
        }

        // We just need to copy this and change the type to match our class.
        @Override
        public MyParcelable[] newArray(int size) {
            return new MyParcelable[size];
        }
    };
}
```

Note that the `Parcelable` interface has two methods defined: `int describeContents()` and `void writeToParcel(Parcel dest, int flags)`. After implementing the `Parcelable` interface, we need to create the `Parcelable.Creator<MyParcelable> CREATOR` constant for our class which requires us to define `createFromParcel`, `newArray`.

### Passing Data Between Intents

We can now pass the parcelable data between activities within an intent:

```java
// somewhere inside an Activity
MyParcelable dataToSend = new MyParcelable();
Intent i = new Intent(this, NewActivity.class);
i.putExtra("myData", dataToSend); // using the (String name, Parcelable value) overload!
startActivity(i); // dataToSend is now passed to the new Activity
```

and then access the data in the NewActivity using:

```java
public class NewActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        MyParcelable object = (MyParcelable) getIntent().getParcelableExtra("myData");
    }
}
```

Now we can access the parcelable data from within the launched activity.

### What It Is Not

You may notice some similarities between `Parcelable` and `Serializable`.  DO NOT, I repeat, DO NOT attempt to persist `Parcel` data.  It is meant for high-performance transport and you could lose data by trying to persist it.

Using `Parcelable` compared to `Serializable` can achieve up to 10x performance increase in many cases for transport which is why it's the Android preferred method. 

### Caveats

There are a few common gotchas associated to Parcelable to consider below:

* One very important thing to pay close attention to is the order that you write and read your values to and from the Parcel.  They need to match up in both cases.  In my example, I write the `int` and then the `String` to the Parcel.  Afterwards, I read them in that same exact order.  The mechanism that Android uses to read the Parcel is blind and completely trusts you to get the order correct, or else you will run into run-time crashes.

* Another problem I have encountered is with `ClassNotFound` exceptions.  This is an issue with the Classloader not finding your class.  To fix this you can manually set the Classloader to use.  If nothing is set, then it will try the default Classloader which leads to the exception. 

* As mentioned before you can only put primitives, lists and arrays, Strings, and other Parcelable objects into a Parcel.  This means that you cannot store framework dependent objects that are not Parcelable.  For example, you could not write a `Drawable` to a Parcel.  To work around this problem, you can instead do something like writing the resource ID of the Drawable as an integer to the Parcel.  On the receiving side you can try to rebuild the Drawable using that.  Remember, Parcel is supposed to be fast and lightweight! (though it is interesting to see `Bitmap` implementing Parcelable)

* Where is `boolean`!?  For whatever odd reason there is no simple way to write a boolean to a Parcel.  To do so, you can instead write a `byte` with the corresponding value with `out.writeByte((byte) (myBoolean ? 1 : 0));` and retrieve it similarly with `myBoolean = in.readByte() != 0;`

### Creating a Parcelable, The Easier Way (using IntelliJ or Android Studio)

There is a [Parcelable plugin](https://github.com/mcharmas/android-parcelable-intellij-plugin) that can be imported directly into IntelliJ or Android Studio, which enables you to generate the boilerplate code for creating Parcelables.  You can install this plugin by going to `Android Studio` -> `Preferences` -> `Plugins` -> `Browse repositories`:

<a href="http://i.imgur.com/jceThxd.gif" alt="Installing Parcelable"><img src="http://i.imgur.com/jceThxd.gif"></a>

Here are all the Java types it supports:

 * Types implementing Parcelable
 * Custom support (avoids `Serializable`/`Parcelable` implementation) for: `Date`, `Bundle`
 * Types implementing Serializable
 * List of `Parcelable` objects
 * Enumerations
 * Primitive types: `long`, `int`, `float`, `double`, `boolean`, `byte`, `String`
 * Primitive type wrappers (written with `Parcel.writeValue(Object)`): `Integer`, `Long`, `Float`, `Double`, `Boolean`, `Byte`
 * Primitive type arrays: `boolean[]`, `byte[]`, `char[]`, `double[]`, `float[]`, `int[]`, `long[]`
 * List type of any object (**Warning: validation is not performed**)

<img src="https://github.com/mcharmas/android-parcelable-intellij-plugin/raw/master/screenshot.png"/>

### Creating a Parcelable, The Easier Way (using Eclipse)

Generate your normal class like so:

```java
public class MyCustomObject {
	
	private int num1;
	private String string1;
	private boolean bool1;
	
	public MyCustomObject() {
		
	}

	public MyCustomObject(int num1, String string1, boolean bool1) {
		super();
		this.num1 = num1;
		this.string1 = string1;
		this.bool1 = bool1;
	}

	public int getNum1() {
		return num1;
	}

	public void setNum1(int num1) {
		this.num1 = num1;
	}

	public String getString1() {
		return string1;
	}

	public void setString1(String string1) {
		this.string1 = string1;
	}

	public boolean isBool1() {
		return bool1;
	}

	public void setBool1(boolean bool1) {
		this.bool1 = bool1;
	}
	
}
```

Then go to the site [Parcelabler](http://devk.it/proj/parcelabler/) designed by Dallas Gutauckis.
Paste your object into the "Code" text box provided.
Then watch the magic happen as this site turns your once plain and non-passable object into.

```java 
public class MyCustomObject implements Parcelable {
	
	private int num1;
	private String string1;
	private boolean bool1;
	
	public MyCustomObject() {
		
	}

	public MyCustomObject(int num1, String string1, boolean bool1) {
		super();
		this.num1 = num1;
		this.string1 = string1;
		this.bool1 = bool1;
	}

	public int getNum1() {
		return num1;
	}

	public void setNum1(int num1) {
		this.num1 = num1;
	}

	public String getString1() {
		return string1;
	}

	public void setString1(String string1) {
		this.string1 = string1;
	}

	public boolean isBool1() {
		return bool1;
	}

	public void setBool1(boolean bool1) {
		this.bool1 = bool1;
	}
	

    protected MyCustomObject(Parcel in) {
        num1 = in.readInt();
        string1 = in.readString();
        bool1 = in.readByte() != 0x00;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(num1);
        dest.writeString(string1);
        dest.writeByte((byte) (bool1 ? 0x01 : 0x00));
    }

    @SuppressWarnings("unused")
    public static final Parcelable.Creator<MyCustomObject> CREATOR = new Parcelable.Creator<MyCustomObject>() {
        @Override
        public MyCustomObject createFromParcel(Parcel in) {
            return new MyCustomObject(in);
        }

        @Override
        public MyCustomObject[] newArray(int size) {
            return new MyCustomObject[size];
        }
    };
}
```
A very pretty Parcelable object that you can copy and paste back into your project.
The site is also able to handle all of the primitives and Java object I didn't include in this tutorial. On top of that it also can handle your own custom parcelable objects and objects that extend other objects that you have already created. This is a util that I stumbled upon searching for a easier way to create Parcelable objects that I can use in my projects. Hope you enjoy.

## References

* <http://www.developerphil.com/parcelable-vs-serializable/>
* <http://mobile.dzone.com/articles/using-android-parcel>
* <http://developer.android.com/reference/android/os/Parcelable.html>
* <http://developer.android.com/reference/android/os/Parcel.html>
* <http://stackoverflow.com/questions/6201311/how-to-read-write-a-boolean-when-implementing-the-parcelable-interface>
* <http://devk.it/proj/parcelabler/>
* <https://github.com/mcharmas/android-parcelable-intellij-plugin>
* <http://shri.blog.kraya.co.uk/2010/04/26/android-parcel-data-to-pass-between-activities-using-parcelable-classes/>