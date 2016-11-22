## Overview

One of the benefits of using [[RxJava|RxJava]] is the ability to compose asynchronous events in a more straightforward way.  Complex UI interactions in Android can also be greatly simplified too, especially when there are multiple UI events that can trigger.  To describe the interaction pattern with standard Android code, you would usually need to use a combination of both `listeners`, `Handlers` and `AsyncTask` (see [this ebook](http://www.oreilly.com/programming/free/files/rxjava-for-android-app-development.pdf) for an example).  With `RxJava` and `RxBinding`, this logic can be greatly simplified and you can describe the interaction patterns between multiple UI components.

## Setup

Make sure to follow the [[setup instructions|RxJava#setup]] for RxJava first.

To use RxBinding in your project, add the following line to your module level `build.gradle` file:

```gradle 
compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'
```

If you rely on any widgets from the 'support-v4' library bindings, make sure to add the following:

```gradle 
compile 'com.jakewharton.rxbinding:rxbinding-support-v4:0.4.0'
```

You may also need to add any widgets that rely on the 'appcompat-v7' library:

```gradle 
compile 'com.jakewharton.rxbinding:rxbinding-appcompat-v7:0.4.0'
```

Check out this [link](https://github.com/JakeWharton/RxBinding) to learn more about `RxBinding`.

### Converting to Observables

You can use the RxJava to convert Android view events to Observables.  Essentially the library wraps the listener and implements the `Observable` interface as described in [[this section|RxJava#defining-observables]].

Normally we react to a click event on Android like the following example:

```java
Button button = (Button)findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View v) {
      //handle on click here
   }
});
```

Using `RxBinding`, same can be accomplished with `RxJava` subscription:

```java
Button button = (Button)findViewById(R.id.button);
Subscription buttonSub = RxView.clicks(button).subscribe(new Action1<Void>() {
   @Override
   public void call(Void aVoid) {
      //handle on click here
   }
}); 

// make sure to unsubscribe the subscription.
```
Another example handling text change on an EditText. Vanila android:
``` java 
EditText editText = (EditText)findViewById(R.id.editText);
editText.addTextChangedListener(new TextWatcher() {
   @Override
   public void beforeTextChanged(CharSequence s, int start, int count, int after) {
   
   }
   
   @Override
   public void onTextChanged(CharSequence s, int start, int before, int count) {
      // do some work with new text
   }
   
   @Override
   public void afterTextChanged(Editable s) {
   
   }
});
```

Same thing written with RxBinding support:
```java 
EditText editText = (EditText)findViewById(R.id.editText);
Subscription editTextSub = RxTextView.textChanges(editText).subscribe(new Action1<CharSequence>() {
   @Override
   public void call(CharSequence value) {
      // do some work with new text
   }
});

// make sure to unsubscribe the subscription.
```

#### Example

For example, consider what you would need to do to implement a `SearchView` that will trigger an API call to fetch results related to the search term, so long as the term is at least three characters long and there has been at least a 100 milliseconds delay since the user last modified the search term in the view.   With RxJava, this pattern can be described as follows:

```java 
RxTextView.textChanges(searchTextView)
	.filter(new Func1<String, Boolean> (){
		@Override
		public Boolean call(String s) {
			return s.length() > 2;
		}
	})
	.debounce(100, TimeUnit.MILLISECONDS)
	.switchMap(new Func1<String, Observable<List<Result>>>() {
		makeApiCall(s);
	})
	.subscribeOn(Schedulers.io())
	.observeOn(AndroidSchedulers.mainThread())
	.subscribe(/* attach observer */);
```

The `filter` operation will only trigger if there is more than 2 characters, the `debounce` operation will filter multiple events that occur within 100 ms, and the `switchMap` operation will return an API response in the form of another `Observable` to make the API call. SwitchMap is preferred over FlatMap because it ignores stale response - http://stackoverflow.com/questions/28175702/what-is-the-difference-between-flatmap-and-switchmap-in-rxjava

## Attribution

This guide was originally drafted by [Adegeye Mayowa](https://github.com/mayojava).

### References

* <http://www.oreilly.com/programming/free/files/rxjava-for-android-app-development.pdf>
