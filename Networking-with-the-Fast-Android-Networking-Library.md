### About Fast Android Networking Library

[Fast Android Networking](https://github.com/amitshekhariitbhu/Fast-Android-Networking) Library is a powerful library for doing any type of networking in Android applications which is made on top of [OkHttp Networking Layer](http://square.github.io/okhttp/).

Fast Android Networking Library takes care of each and everything. So you don't have to do anything, just make the request and listen for the response. This library provides you the simplest way to do any type of networking in android including downloading files, uploading files, consuming REST API, etc.

## Requirements

Fast Android Networking Library can be included in any Android application. 

Fast Android Networking Library supports Android 2.3 (Gingerbread) and later. 

## Using Fast Android Networking Library in your application

Add this in your build.gradle
```groovy
compile 'com.amitshekhar.android:android-networking:1.0.0'
```

### For RxJava2 Support, [check here](https://amitshekhariitbhu.github.io/Fast-Android-Networking/rxjava2_support.html).

Do not forget to add internet permission in manifest if already not present
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
Then initialize it in onCreate() Method of application class :
```java
AndroidNetworking.initialize(getApplicationContext());
```
### Making a GET Request
```java
AndroidNetworking.get("https://fierce-cove-29863.herokuapp.com/getAllUsers/{pageNumber}")
                 .addPathParameter("pageNumber", "0")
                 .addQueryParameter("limit", "3")
                 .addHeaders("token", "1234")
                 .setTag("test")
                 .setPriority(Priority.LOW)
                 .build()
                 .getAsJSONArray(new JSONArrayRequestListener() {
                    @Override
                    public void onResponse(JSONArray response) {
                      // do anything with response
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error
                    }
                });                
```
### Making a POST Request
```java
AndroidNetworking.post("https://fierce-cove-29863.herokuapp.com/createAnUser")
                 .addBodyParameter("firstname", "Amit")
                 .addBodyParameter("lastname", "Shekhar")
                 .setTag("test")
                 .setPriority(Priority.MEDIUM)
                 .build()
                 .getAsJSONObject(new JSONObjectRequestListener() {
                    @Override
                    public void onResponse(JSONObject response) {
                      // do anything with response
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error
                    }
                });
```
You can also post java object, json, file, etc in POST request like this.
```java
User user = new User();
user.firstname = "Amit";
user.lastname = "Shekhar";

AndroidNetworking.post("https://fierce-cove-29863.herokuapp.com/createUser")
                 .addBodyParameter(user) // posting java object
                 .setTag("test")
                 .setPriority(Priority.MEDIUM)
                 .build()
                 .getAsJSONArray(new JSONArrayRequestListener() {
                    @Override
                    public void onResponse(JSONArray response) {
                      // do anything with response
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error
                    }
                });

JSONObject jsonObject = new JSONObject();
try {
    jsonObject.put("firstname", "Rohit");
    jsonObject.put("lastname", "Kumar");
} catch (JSONException e) {
  e.printStackTrace();
}
       
AndroidNetworking.post("https://fierce-cove-29863.herokuapp.com/createUser")
                 .addJSONObjectBody(jsonObject) // posting json
                 .setTag("test")
                 .setPriority(Priority.MEDIUM)
                 .build()
                 .getAsJSONArray(new JSONArrayRequestListener() {
                    @Override
                    public void onResponse(JSONArray response) {
                      // do anything with response
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error
                    }
                });
                
AndroidNetworking.post("https://fierce-cove-29863.herokuapp.com/postFile")
                 .addFileBody(file) // posting any type of file
                 .setTag("test")
                 .setPriority(Priority.MEDIUM)
                 .build()
                 .getAsJSONObject(new JSONObjectRequestListener() {
                    @Override
                    public void onResponse(JSONObject response) {
                      // do anything with response
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error
                    }
                });               
```

### Using it with your own JAVA Object - JSON Parser
```java
/*--------------Example One -> Getting the userList----------------*/
AndroidNetworking.get("https://fierce-cove-29863.herokuapp.com/getAllUsers/{pageNumber}")
                .addPathParameter("pageNumber", "0")
                .addQueryParameter("limit", "3")
                .setTag(this)
                .setPriority(Priority.LOW)
                .build()
                .getAsObjectList(User.class, new ParsedRequestListener<List<User>>() {
                    @Override
                    public void onResponse(List<User> users) {
                      // do anything with response
                      Log.d(TAG, "userList size : " + users.size());
                      for (User user : users) {
                        Log.d(TAG, "id : " + user.id);
                        Log.d(TAG, "firstname : " + user.firstname);
                        Log.d(TAG, "lastname : " + user.lastname);
                      }
                    }
                    @Override
                    public void onError(ANError anError) {
                     // handle error
                    }
                });
/*--------------Example Two -> Getting an user----------------*/
AndroidNetworking.get("https://fierce-cove-29863.herokuapp.com/getAnUserDetail/{userId}")
                .addPathParameter("userId", "1")
                .setTag(this)
                .setPriority(Priority.LOW)
                .build()
                .getAsObject(User.class, new ParsedRequestListener<User>() {
                     @Override
                     public void onResponse(User user) {
                        // do anything with response
                        Log.d(TAG, "id : " + user.id);
                        Log.d(TAG, "firstname : " + user.firstname);
                        Log.d(TAG, "lastname : " + user.lastname);
                     }
                     @Override
                     public void onError(ANError anError) {
                        // handle error
                     }
                 }); 
/*-- Note : YourObject.class, getAsObject and getAsObjectList are important here --*/              
```

### Downloading a file from server
```java
AndroidNetworking.download(url,dirPath,fileName)
                 .setTag("downloadTest")
                 .setPriority(Priority.MEDIUM)
                 .build()
                 .setDownloadProgressListener(new DownloadProgressListener() {
                    @Override
                    public void onProgress(long bytesDownloaded, long totalBytes) {
                      // do anything with progress  
                    }
                 })
                 .startDownload(new DownloadListener() {
                    @Override
                    public void onDownloadComplete() {
                      // do anything after completion
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error    
                    }
                });                 
```
### Uploading a file to server
```java
AndroidNetworking.upload(url)
                 .addMultipartFile("image",file)    
                 .addMultipartParameter("key","value")
                 .setTag("uploadTest")
                 .setPriority(Priority.HIGH)
                 .build()
                 .setUploadProgressListener(new UploadProgressListener() {
                    @Override
                    public void onProgress(long bytesUploaded, long totalBytes) {
                      // do anything with progress 
                    }
                 })
                 .getAsJSONObject(new JSONObjectRequestListener() {
                    @Override
                    public void onResponse(JSONObject response) {
                      // do anything with response                
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error 
                    }
                 }); 
```

### Getting Response and completion in an another thread executor 
(Note : Error and Progress will always be returned in main thread of application)
```java
AndroidNetworking.upload(url)
                 .addMultipartFile("image",file)  
                 .addMultipartParameter("key","value")  
                 .setTag("uploadTest")
                 .setPriority(Priority.HIGH)
                 .build()
                 .setExecutor(Executors.newSingleThreadExecutor()) // setting an executor to get response or completion on that executor thread
                 .setUploadProgressListener(new UploadProgressListener() {
                    @Override
                    public void onProgress(long bytesUploaded, long totalBytes) {
                      // do anything with progress 
                    }
                 })
                 .getAsJSONObject(new JSONObjectRequestListener() {
                    @Override
                    public void onResponse(JSONObject response) {
                      // below code will be executed in the executor provided
                      // do anything with response                
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error 
                    }
                 }); 
```
### Getting Bitmap from url with some specified parameters
```java
AndroidNetworking.get(imageUrl)
                 .setTag("imageRequestTag")
                 .setPriority(Priority.MEDIUM)
                 .setBitmapMaxHeight(100)
                 .setBitmapMaxWidth(100)
                 .setBitmapConfig(Bitmap.Config.ARGB_8888)
                 .build()
                 .getAsBitmap(new BitmapRequestListener() {
                    @Override
                    public void onResponse(Bitmap bitmap) {
                    // do anything with bitmap
                    }
                    @Override
                    public void onError(ANError error) {
                      // handle error
                    }
                });
```
### Error Code Handling
```java
public void onError(ANError error) {
   if (error.getErrorCode() != 0) {
        // received error from server
        // error.getErrorCode() - the error code from server
        // error.getErrorBody() - the error body from server
        // error.getErrorDetail() - just an error detail
        Log.d(TAG, "onError errorCode : " + error.getErrorCode());
        Log.d(TAG, "onError errorBody : " + error.getErrorBody());
        Log.d(TAG, "onError errorDetail : " + error.getErrorDetail());
        // get parsed error object (If ApiError is your class)
        ApiError apiError = error.getErrorAsObject(ApiError.class);
   } else {
        // error.getErrorDetail() : connectionError, parseError, requestCancelledError
        Log.d(TAG, "onError errorDetail : " + error.getErrorDetail());
   }
}
```

