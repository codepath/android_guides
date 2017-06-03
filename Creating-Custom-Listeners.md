## Overview

The "listener" or "observer" pattern is the most common strategy for creating asynchronous callbacks within Android development. Listeners are used for any type of asynchronous event in order to implement the code to run when an event occurs. We see this pattern with any type of I/O as well as for view events on screen. For example, here's a common usage of the listener pattern to attach a click event to a button:

```java
Button btnExample = (Button) findViewById(R.id.btnExample);
btnExample.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // Do something here    
    }
});
```

This listener is built-in but we can also create our own listeners and attach callbacks to the events they fire from other areas in our code. This is useful in a variety of cases including:

 * Firing events from list items upwards to an activity from within an adapter
 * Firing events from [[a fragment upwards to an activity|Creating-and-Using-Fragments#fragment-listener]].
 * Firing async events from an abstraction (i.e networking library) to a parent handler.

In short, a listener is useful **anytime a child object** wants to **emit events** upwards to notify a parent object and allow that object to respond.

### Why Listeners?

Listeners are a powerful mechanism for properly separating concerns in your code. One of the essential principles around writing maintainable code is to reduce coupling and complexity using proper encapsulation. Listeners are about ensuring that code is properly organized into the correct places. In particular, this table below offers guidelines about where different types of code should be called:

| Type            | Called By             | Description |
| ----            | --------           | --------------------------------------------------              |
| Intents         | `Activity`           | Intents should be created and executed within activities.     | 
| Networking    | `Activity`, `Fragment` | Networking code should invoked in an activity or fragment.    |
| FragmentManager | `Activity`           | Fragment changes should be invoked by an activity.            |
| Persistence   | `Activity`, `Fragment` | Writing to disk should be invoked by activity or fragment.    |

While there are exceptions, generally speaking this table helps to provide guidelines as to when you need a listener to propagate an event to the appropriate owner. For example, if you are inside an adapter and you **want to launch a new activity**, this is best done by firing an event using a custom listener triggering the parent activity. 

## Custom Listeners

There are four steps to using a custom listener to manage callbacks in your code:

1. **Define an interface** as an event contract with methods that define events and arguments which are relevant event data.
         
2. **Setup a listener member variable** and setter in the child object which can be assigned an implementation of the interface.
         
3. **Owner passes in a listener** which implements the interface and handles the events from the child object.

4. **Trigger events on the defined listener** when the object wants to communicate events to it's owner

The sample code below will demonstrate this process step-by-step.

### 1. Defining the Interface

First, we define an interface in the child object. This object can be a plain java object, an `Adapter`, a `Fragment`, or any object created by a "parent" object such as an activity which will handle the events that are triggered. 

This involves creating an interface class and defining the events that will be fired up to the parent. We need to design the events as well as the data passed when the event is triggered.

```java
public class MyCustomObject {
  // Step 1 - This interface defines the type of messages I want to communicate to my owner  
  public interface MyCustomObjectListener {
      // These methods are the different events and 
      // need to pass relevant arguments related to the event triggered
      public void onObjectReady(String title);
      // or when data has been loaded
      public void onDataLoaded(SomeData data);
  }
}
```

With the interface defined, we need to setup a listener variable to store a particular implementation of the callbacks which will be defined by the owner.

### 2. Create Listener Setter

In the child object, we need to define an instance variable for an implementation of the listener:

```java
public class MyCustomObject {
    // ...

    // Step 2 - This variable represents the listener passed in by the owning object
    // The listener must implement the events interface and passes messages up to the parent.
    private MyCustomObjectListener listener;
}
```

as well as a "setter" which allows the listener callbacks to be defined in the parent object:

```java
public class MyCustomObject {
    // Constructor where listener events are ignored
    public MyCustomObject() {
        // set null or default listener or accept as argument to constructor
        this.listener = null; 
    }
	
    // Assign the listener implementing events interface that will receive the events
    public void setCustomObjectListener(MyCustomObjectListener listener) {
        this.listener = listener;
    }
}
```

This allows the parent object to pass in an implementation of the listener after this child object is created similar to how the button accepts the click listener.

### 3. Implement Listener Callback

Now that we have created the setters, the parent object can construct and set callbacks for the child object:

```java
public class MyParentActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
    	// ...
    	// Create the custom object
    	MyCustomObject object = new MyCustomObject();
    	// Step 4 - Setup the listener for this object
    	object.setCustomObjectListener(new MyCustomObject.MyCustomObjectListener() {
    		@Override
    		public void onObjectReady(String title) {
    		    // Code to handle object ready
    		}
    		
    		@Override
    		public void onDataLoaded(SomeData data) {
    		    // Code to handle data loaded from network
    		    // Use the data here!
    		}
    	});
    }
}
```

Here we've created the child object and passed in the callback implementation for the listener. These events can now be fired by the child object to pass the event to the parent as appropriate.

### 4. Trigger Listener Events

Now the child object should trigger events to the listener whenever appropriate and pass along the data to the parent object through the event. These events can be triggered when a user action is taken or when an asynchronous task such as networking or persistence is completed. For example, we will trigger the `onDataLoaded(SomeData data)` event once this network request comes back on the child:

```java
public class MyCustomObject {
   // Listener defined earlier
   public interface MyCustomObjectListener {
      public void onObjectReady(String title);
      public void onDataLoaded(SomeData data);
   }
   
   // Member variable was defined earlier
   private MyCustomObjectListener listener;

    // Constructor where listener events are ignored
    public MyCustomObject() {
        // set null or default listener or accept as argument to constructor
        this.listener = null; 
        loadDataAsync();
    }
  
    // ... setter defined here as shown earlier

    public void loadDataAsync() {
        AsyncHttpClient client = new AsyncHttpClient();
        client.get("https://mycustomapi.com/data/get.json", new JsonHttpResponseHandler() {         
            @Override
            public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
                // Networking is finished loading data, data is processed
                SomeData data = SomeData.processData(response.get("data"));
                // Do some other stuff as needed....
                // Now let's trigger the event
                if (listener != null)
                  listener.onDataLoaded(data); // <---- fire listener here
            }
        });
    }
}
```

Notice that we fire the listener event `listener.onDataLoaded(data)` when the asynchronous network request has completed. Whichever callback has been passed by the parent (shown in the previous step) will be fired.

### Conclusion

The "listener pattern" is a very powerful Java pattern that can be used to emit events to a single parent in order to communicate important information asynchronously. This can be used to **move complex logic out of adapters**, **create useful abstractions for your code** or **communicate from a fragment to your activities**. 

## Callback Approaches

There are several different ways to pass a listener callback into the child object:

1. Pass the callback through a method call
2. Pass the callback through the constructor
3. Pass the callback through a lifecycle event

### 1. Passing via Method

The code earlier demonstrated passing the callback through a method call like this:

```java
// Inside the parent object
childObject.setCustomObjectListener(new MyCustomObject.MyCustomObjectListener() {
    @Override
    public void onObjectReady(String title) {
        // Code to handle object ready
    }
});
```

which is defined in the child object as follows:

```java
// Inside the child object
private MyCustomObjectListener listener;

// Assign the listener implementing events interface that will receive the events
public void setCustomObjectListener(MyCustomObjectListener listener) {
    this.listener = listener;
}
```

### 2. Passing via Constructor

If the callback is critical to the object's function, we can pass the callback directly into the constructor of the child object as an argument:

```java
// Inside the child object
private MyCustomObjectListener listener;

// Passing in the listener directly into the constructor
public MyCustomObject(MyCustomObjectListener listener) {
    this.listener = listener; 
}
```

and then when we can create the child object from the parent we can do the following:

```java
// Inside the parent object
MyCustomObject object = new MyCustomObject(new MyCustomObject.MyCustomObjectListener() {
    @Override
    public void onObjectReady(String title) {
        // Code to handle object ready
    });
});
```

### 3. Pass via Lifecycle Event

The third case is most common with fragments or other android components that need to communicate upward to a parent. In this case, we can leverage existing Android lifecycle events to get access to the listener. In the case below, a fragment being attached to an activity:

```java
public class MyListFragment extends Fragment {
  // Member variable storing the listener
  private MyCustomObjectListener listener;

  // Store the listener that will have events fired once the fragment is attached
  @Override
  public void onAttach(Context context) {
    super.onAttach(context);
    if (context instanceof MyCustomObjectListener) {
        listener = (MyCustomObjectListener) context;
    } else {
        throw new ClassCastException(context.toString()
            + " must implement MyListFragment.MyCustomObjectListener");
    }
  }
}
```

For the full details on this approach, check out the [[fragments guide|Creating-and-Using-Fragments#fragment-listener]].

## References 

* <https://gist.github.com/nesquena/8265f057fef203a2c67e>
* <http://www.vogella.com/tutorials/DesignPatternObserver/article.html>