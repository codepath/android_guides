# Overview

Volley is a library that makes networking for Android apps easier and most importantly, faster.

Volley Library was announced by [Ficus Kirkpatrick](https://plus.google.com/+FicusKirkpatrick) at Google I/O '13!
It was first used by the Play Store team in Play Store Application and then they released it as an Open Source Library.

***

# Why Volley?

* Volley can pretty much do everything with that has to do with Networking in Android.
It has some built-in Requests, such as JsonObjectRequest-JsonArrayRequest-StringRequest etc, based on the generic type of the Request<T> class inside the Library. 

* Volley automatically schedule all network requests. It means that Volley will be taking care of all the network requests your app executes for fetching response or image from web.

* Volley provides transparent disk and memory caching.

* Volley provides powerful cancellation request API. It means that you can cancel a single request or you can set blocks or scopes of requests to cancel.

* Volley provides powerful customization abilities.

* Volley provides Debugging and tracing tools

# Getting started

1. First of all you have to download volley by cloning the project from `git clone https://android.googlesource.com/platform/frameworks/volley`

2. After you have done this you just copy the **com** folder inside your package and start using Volley!

# How to use Volley?

Volley has two Main Class files that you will have to deal with basically.

1. RequestQueue
2. Request (and any extension of it)

In your Activity's onCreate Method or in any other Java file you want you just create a new RequestQueue object like in the example below:

`public YourActivity extends Activty{

	private RequestQueue mRequestQueue;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main_screen_layout);
		
		mRequestQueue = Volley.newRequestQueue(this);
		
	}
}`

After this step you are ready to create your Request object or some extension of it: 

**Needs Attention**

# References

* <http://java.dzone.com/articles/android-%E2%80%93-volley-library>