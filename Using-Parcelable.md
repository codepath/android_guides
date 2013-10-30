## Overview

Due to Android's memory management scheme, you will often find yourself needing to communicate with different components of your application, system components, or other applications installed on the phone.  Parcelable will help you pass data between these components.  Android uses what is called the Binder to facilitate such communication in a highly optimized way.  The Binder communicates with Parcels, which is a message container.  The Binder marshalls the Parcel to be sent, sends and receives it, and then unmarshalls it on the other side to reconstruct a copy of the original Parcel.  

To allow for your class instances to be sent as a Parcel you must implement the Parcelable interface along with a static field called CREATOR, which itself requires a special constructor in your class.

Here is a typical implementation:

```java
public class MyParcelable implements Parcelable {
    private int mData;
    private String mName;

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(mData);
        out.writeString(mName);
    }

    public static final Parcelable.Creator<MyParcelable> CREATOR
            = new Parcelable.Creator<MyParcelable>() {
        @Override
        public MyParcelable createFromParcel(Parcel in) {
            return new MyParcelable(in);
        }

        @Override
        public MyParcelable[] newArray(int size) {
            return new MyParcelable[size];
        }
    };
     
    private MyParcelable(Parcel in) {
        mData = in.readInt();
        mName = in.readString();
    }
}
```

The `Parcelable` interface has two methods defined: `int describeContents()` and `void writeToParcel(Parcel dest, int flags)`:

* `int describeContents()` 

In the vast majority of cases you can simply return 0 for this.  There are cases where you need to use the constant `CONTENTS_FILE_DESCRIPTOR`, also defined in the interface, in combination with this method but it is very rare and out of the scope of this tutorial.  

* `void writeToParcel(Parcel dest, int flags)`

This is where you write the values you want to save to the `Parcel`.  The `Parcel` class has methods defined to help you save all of your values.  Note that there are only methods defined for primitive values (and String!), lists and arrays of primitive values, and other Parcelable objects.  You may need to make several classes Parcelable to send the data you want.

After implementing the `Parcelable` interface, we need to create the `Parcelable.Creator<MyParcelable> CREATOR` constant for our class; notice how it has our class as its type.  This `CREATOR` object also has two methods:

* `MyParcelable createFromParcel(Parcel in)`

This simply calls our new constructor (typically private) and passes along the unmarshalled `Parcel`, and then returns the new object!

* `MyParcelable[] newArray(int size)`

We just need to copy this and change the type to match our class.

Lastly, we have the private constructor that we created:

* `MyParcelable(Parcel in)`

Using the `in` variable, we can retrieve the values that we originally wrote into the `Parcel`.  This constructor is usually private so that only the `CREATOR` field can access it and to not clutter your exposed API.

### Caveats

* One very important thing to pay close attention to is the order that you write and read your values to and from the Parcel.  They need to match up in both cases.  In my example, I write the `int` and then the `String` to the Parcel.  Afterwards, I read them in that same exact order.  The mechanism that Android uses to read the Parcel is blind and completely trusts you to get the order correct, or else you will run into run-time crashes.

* Another problem I have encountered is with `ClassNotFound` exceptions.  This is an issue with the Classloader not finding your class.  To fix this you can manually set the Classloader to use.  If nothing is set, then it will try the default Classloader which leads to the exception. 

* As mentioned before you can only put primitives, lists and arrays, Strings, and other Parcelable objects into a Parcel.  This means that you cannot store framework dependent objects that are not Parcelable.  For example, you could not write a `Drawable` to a Parcel.  To work around this problem, you can instead do something like writing the resource ID of the Drawable as an integer to the Parcel.  On the receiving side you can try to rebuild the Drawable using that.  Remember, Parcel is supposed to be fast and lightweight! (though it is interesting to see `Bitmap` implementing Parcelable)

* Where is `boolean`!?  For whatever odd reason there is no simple way to write a boolean to a Parcel.  To do so, you can instead write a `byte` with the corresponding value like so:

`out.writeByte((byte) (myBoolean ? 1 : 0));`

And retrieve it similarly:

`myBoolean = in.readByte() != 0;`

### What It Is Not

You may notice some similarities between `Parcelable` and `Serializable`.  DO NOT, I repeat, DO NOT attempt to persist `Parcel` data.  It is meant for high-performance transport and you could lose data by trying to persist it.

Using `Parcelable` compared to `Serializable` can achieve up to 10x performance increase in many cases for transport which is why it's the Android preferred method. :)

## Conclusion

You should now be confident in making your classes `Parcelable`.  Taking our earlier example code, we can now achieve something like this:

```java
//somewhere inside an Activity
MyParcelable dataToSend = new MyParcelable();
Intent i = new Intent(this, NewActivity.class);
i.putExtra("myData", dataToSend); //using the (String name, Parcelable value) overload!
startActivity(i); //dataToSend is now passed to the new Activity
```

## References

* <http://www.developerphil.com/parcelable-vs-serializable/>
* <http://mobile.dzone.com/articles/using-android-parcel>
* <http://developer.android.com/reference/android/os/Parcelable.html>
* <http://developer.android.com/reference/android/os/Parcel.html>
* <http://stackoverflow.com/questions/6201311/how-to-read-write-a-boolean-when-implementing-the-parcelable-interface>