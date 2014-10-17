## Overview

This guide describes the process of converting JSON data retrieved from a network request and converting that data to simple Model objects. This approach will be compatible with nearly any basic data-driven application and fits well with any ORM solution for persistence such as [ActiveAndroid](http://www.activeandroid.com/) or [SugarORM](http://satyan.github.io/sugar/) that may be introduced.

For this guide, we will be using the [Yelp API](http://www.yelp.com/developers/documentation/v2/search_api#searchGP) as the example. The goal of this guide is to **perform a Yelp API Search** and then **process the results into Java objects** which we can then use to populate information within our application.

The model in this case is **Business** and for our application, let's suppose we just need the _name_, _phone_, and _image_ of the business which are all provided by the [Search API](http://www.yelp.com/developers/documentation/v2/search_api#searchGP).

### Fetching JSON Results

The first step in retrieving any API-based model data is to execute a network request to retrieve the JSON response that represents the underlying data that we want to use. In this case, we want to execute a request to `http://api.yelp.com/v2/search?term=food&location=San+Francisco` and then this will return us a JSON dictionary that looks like:

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
	public void onSuccess(int code, JSONObject body) {
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
      ArrayList<Business> businesses = new ArrayList<Business>(jsonArray.length());
      // Process each result in json array, decode and convert to business object
      for (int i=0; i < jsonArray.length(); i++) {
          JSONObject businessJson = null;
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

Now, we can return to our Activity where we are executing the network request and use the new Model to get easy access to our Business data. Let's tweak the network request handler:

```java
YelpClient client = YelpClientApp.getRestClient();
client.search("food", "san francisco", new JsonHttpResponseHandler() {
  @Override
  public void onSuccess(int code, JSONObject body) {
    try {
      JSONArray businessesJson = body.getJSONArray("businesses");
      ArrayList<Business> businesses = Business.fromJson(businessesJson);
      // Now we have an array of business objects
      // Might now create an ArrayAdapter<Business> to load the businesses into a ListView
    } catch (JSONException e) {
      e.printStackTrace();
    }
  }

  @Override
  public void onFailure(Throwable arg0) {
    Toast.makeText(getBaseContext(), "FAIL", Toast.LENGTH_LONG).show();
  }
});
```

This approach works very similarly for any simple API data which often is returned in collections whether it be images on Instagram, tweets on Twitter, or auctions on Ebay. The next step might be to create an `ArrayAdapter<Business>` and populate these new model objects into a ListView.