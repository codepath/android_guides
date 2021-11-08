{
    "statues": "200",
    "msg": "تم تسجيل الدخول بنجاح",
    "data": {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9hcHAuamFkZWVlci5jb21cL1wvYXBpXC92MVwvbG9naW4iLCJpYXQiOjE2MzYyOTE1MDgsIm5iZiI6MTYzNjI5MTUwOCwianRpIjoiZW5PVXR5WERBamRUZ0hWSCIsInN1YiI6MjQsInBydiI6IjM3ODdmYmExNjE4YTkzMDUyNmFjYTZjOGJiOWI0NGI4M2YyOTc3MjYifQ.74guBdv6DJSwp5Sa8U98B0wjTxGB9TSv9nLpbsqhy1Q",
        "data": {
            "id": 24,
            "first_name": "محمد",
            "last_name": "المالكي",
            "full_name": "محمد المالكي",
            "email": "",
            "phone": "0557002096",
            "gender": "Male",
            "image": "http://graph.facebook.com//picture?type=large&height=500&width=500",
            "api_token": null,
            "district_id": "0",
            "district": null,
            "store": null,
            "car": {
                "id": 17,
                "number": "2322",
                "client_id": "24",
                "Type_car": "1",
                "driver_license": "http://app.jadeeer.com/public/storage/images/car/2416294795623674.jpg",
                "car_license": "http://app.jadeeer.com/public/storage/images/car/2416294795623595.jpg",
                "personal_id": "http://app.jadeeer.com/public/storage/images/car/2416294795623885.jpg",
                "image_car_front": "http://app.jadeeer.com/public/storage/images/car/2416294795631246.jpg",
                "image_car_back": "http://app.jadeeer.com/public/storage/images/car/2416294795658388.jpg",
                "car_model": "19",
                "car_category": null,
                "stc_pay": "0557002096",
                "char_car": "tyur",
                "status": "0",
                "nationality": "2",
                "date_of_birth": null,
                "gender": "Male",
                "activated": "1",
                "bank_name": "الراجحي",
                "name_card": "Mohammed",
                "ipan": "6566666666666666",
                "nationality_id": "2",
                "brand_id": {
                    "id": 2,
                    "Category_id": "22",
                    "name": "هونداي",
                    "Category": {
                        "id": 22,
                        "name": "السيارات والشاحنات",
                        "image": "http://app.jadeeer.com/public/storage/images/category/1613175588.jpg"
                    }
                }
            }
        }
    },
    "count": 0
}

```json
{
  "businesses": [
    {
      "id": "yelp-tropisueno",
      "name" : "Tropisueno",
      "display_phone": "+1-415-908-3801",
      "image_url": "http://s3-media2.ak.yelpcdn.com/bphoto/7DIHu8a0AHhw-BffrDIxPA/ms.jpg",
      ...
    }
  ]
}
```

Sending out this API request can be done in any number of ways but first requires us to register for a Yelp developer account and use OAuth 1.0a to authenticate with our provided access_token. You might for example use our [rest-client-template](https://github.com/codepath/android-rest-client-template) to manage this authentication and then construct a `YelpClient` that has a `search` method:

```java
public class YelpClient extends OAuthBaseClient {
    // LOTS OF TOKENS AND STUFF ...

    // Setting up the search endpoint
    public void search(String term, String location, AsyncHttpResponseHandler handler) {
    	// http://api.yelp.com/v2/search?term=food&location=San+Francisco
    	String apiUrl = getApiUrl("search");
        RequestParams params = new RequestParams();
        params.put("term", term);
        params.put("location", location);
        client.get(apiUrl, params, handler);
    }
}
```

This `search` method will take care of executing our JSON request to the Yelp API. The API call might be executed in an Activity now when the user performs a search. Executing the API request would look like:

```java
YelpClient client = YelpClientApp.getRestClient();
client.search("food", "san francisco", new JsonHttpResponseHandler() {
	@Override
        public void onSuccess(int code, Header[] headers, JSONObject body) {
		try {
			JSONArray businessesJson = body.getJSONArray("businesses");
                        // Here we now have the json array of businesses!
			Log.d("DEBUG", businesses.toString());
		} catch (JSONException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void onFailure(Throwable arg0) {
		Toast.makeText(SearchActivity.this, "FAIL", Toast.LENGTH_LONG).show();
	}
});
```

We could now run the app and verify that the JSON array of business has the format we expect from the provided sample response in the documentation.

### Setting up our Model

The primary resource in the Yelp API is the **Business**. Let's create a Java class that will act as the **Business** model in our application:

```java
public class Business {
	private String id;
	private String name;
	private String phone;
	private String imageUrl;

	public String getName() {
		return this.name;
	}

	public String getPhone() {
		return this.phone;
	}

	public String getImageUrl() {
		return this.imageUrl;
	}
}
```

So far, the business model is just a series of declared fields and then getters to access those fields. Next, we need to add method that would manage the deserialization of a JSON dictionary into a populated Business object:

```java
public class Business {
  // ...

  // Decodes business json into business model object
  public static Business fromJson(JSONObject jsonObject) {
  	Business b = new Business();
        // Deserialize json into object fields
  	try {
  		b.id = jsonObject.getString("id");
        	b.name = jsonObject.getString("name");
        	b.phone = jsonObject.getString("display_phone");
        	b.imageUrl = jsonObject.getString("image_url");
        } catch (JSONException e) {
            e.printStackTrace();
            return null;
        }
  	// Return new object
  	return b;
  }
}
```

With this method in place, we could take a single business JSON dictionary such as:

```json
{
 "id": "yelp-tropisueno",
 "name" : "Tropisueno",
 "display_phone": "+1-415-908-3801",
 "image_url": "http://s3-media2.ak.yelpcdn.com/bphoto/7DIHu8a0AHhw-BffrDIxPA/ms.jpg",
  ...
}
```

and successfully create a Business with `Business.fromJson(json)`. However, in the API response, we actually get a collection of business JSON in an array. So ideally we also would have an easy way of processing an **array of businesses**  into an **ArrayList of Business objects**. That might look like:

```java
public class Business {
  // ...fields and getters
  // ...fromJson for an object

  // Decodes array of business json results into business model objects
  public static ArrayList<Business> fromJson(JSONArray jsonArray) {
      JSONObject businessJson;
      ArrayList<Business> businesses = new ArrayList<Business>(jsonArray.length());
      // Process each result in json array, decode and convert to business object
      for (int i=0; i < jsonArray.length(); i++) {
          try {
          	businessJson = jsonArray.getJSONObject(i);
          } catch (Exception e) {
              e.printStackTrace();
              continue;
          }

          Business business = Business.fromJson(businessJson);
          if (business != null) {
          	businesses.add(business);
          }
      }

      return businesses;
  }
}
```

With that in place, we can now pass an JSONArray of business json data and process that easily into a nice ArrayList<Business> object for easy use in our application with `Business.fromJson(myJsonArray)`.

### Putting It All Together

Now, we can return to our Activity where we are executing the network request and use the new Model to get easy access to our Business data. Let's tweak the network request handler in our activity:

```java
// Within an activity or fragment
YelpClient client = YelpClientApp.getRestClient();
client.search("food", "san francisco", new JsonHttpResponseHandler() {
  @Override
  public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
    try {
      JSONArray businessesJson = body.getJSONArray("businesses");
      ArrayList<Business> businesses = Business.fromJson(businessesJson);
      // Now we have an array of business objects
      // Might now create an adapter BusinessArrayAdapter<Business> to load the businesses into a list
      // You might also simply update the data in an existing array and then notify the adapter
    } catch (JSONException e) {
      e.printStackTrace();
    }
  }

  @Override
  public void onFailure(int statusCode, Header[] headers, String res, Throwable t) {
    Toast.makeText(getBaseContext(), "FAIL", Toast.LENGTH_LONG).show();
  }
});
```

This approach works very similarly for any simple API data which often is returned in collections whether it be images on Instagram, tweets on Twitter, or auctions on Ebay.

### Bonus: Setting Up Your Adapter

The next step might be to create an adapter and populate these new model objects into a `ListView` or `RecyclerView`.

```java
// Within an activity
ArrayList<Business> businesses;
BusinessRecyclerViewAdapter adapter;

@Override
protected void onCreate(Bundle savedInstanceState) {
   // ...
   // Lookup the recyclerview in activity layout
   RecyclerView rvBusinesses = (RecyclerView) findViewById(...);
   // Initialize default business array
   businesses = new ArrayList<Business>();
   // Create adapter passing in the sample user data
   adapter = new BusinessRecyclerViewAdapter(business);
   rvBusinesses.setAdapter(adapter);
   // Set layout manager to position the items
   rvBusinesses.setLayoutManager(new LinearLayoutManager(this));
}

// Anywhere in your activity
client.search("food", "san francisco", new JsonHttpResponseHandler() {
  @Override
  public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
      try {
          // Get and store decoded array of business results
          JSONArray businessesJson = response.getJSONArray("businesses");
          businesses.clear(); // clear existing items if needed
          businesses.addAll(Business.fromJson(businessesJson)); // add new items
          adapter.notifyDataSetChanged();
      } catch (JSONException e) {
          e.printStackTrace();
      }
  }
}
```

For additional details on using adapters to display data in lists, see [[Using RecyclerView|Using-the-RecyclerView#creating-the-recyclerviewadapter]] or [[Using ListView|Using-an-ArrayAdapter-with-ListView]].
