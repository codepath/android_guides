## Overview

Due to Android's memory management scheme, you will often find yourself needing to communicate with remote parts of your application, other applications installed on the phone, or system components.  Android uses what is called the Binder to facilitate such communication in a highly optimized way.  The Binder communicates with Parcels, which is a message container.  The Binder marshalls the Parcel to be sent, sends and receives it, and then unmarshalls it on the other side to reconstruct a copy of the original Parcel.  

To allow for your class instances to be sent as a Parcel you must implement the Parcelable interface along with a static field called CREATOR, which itself requires a special private constructor in your class.

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

This simply calls our new private constructor and passes along the unmarshalled `Parcel`, and then returns the new object!

* `MyParcelable[] newArray(int size)`

We just need to copy this and change the type to match our class.

Lastly, we have the private constructor that we created:

* `MyParcelable(Parcel in)`

Using the `in` variable, we can retrieve the values that we originally wrote into the `Parcel`.

### Caveat

Something this complicated does not come without cautions!  One very big thing to pay close attention to is the order that you write and read your values to and from the Parcel.  They need to match up in both cases.  In my example, I write the `int` and then the `String` to the Parcel.  Afterwards, I read them in that same exact order.  The mechanism that Android uses to read the Parcel is blind and completely trusts you to get the order correct, or else you will run into run-time crashes.

One other problem I have encountered is with `ClassNotFound` exceptions.  This is an issue with the Classloader not finding your class.  To fix this you can manually set the Classloader to use.  If nothing is set, then it will try the default Classloader which leads to the exception. 

### What It Is Not

You may notice some similarities between `Parcelable` and `Serializable`.  DO NOT, I repeat, DO NOT attempt to persist `Parcel` data.  It is meant for high-performance transport and you could lose data by trying to persist it.

Using `Parcelable` compared to `Serializable` can achieve up to 10x performance increase in many cases for transport which is why it's the Android preferred method. :)