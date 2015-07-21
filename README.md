PinterestSDK for Android

The PinterestSDK for Android will allow you to authenticate an account with Pinterest and make requests on behalf of the authenticated user. For details on the supported endpoint, visit [the Pinterest API](https://api.pinterest.com/developers/signup/).

## Installation

### Using AAR

1. Go to Android Studio | New Project | Minimum SDK

2. Select "API 11: Android 3.0" or higher and create your new project.

3. After you create a new project, go to File > New > New Module

4. Select "Import .JAR or >AAR Package"

5. Enter the path to .AAR file downloaded from this repo.

6. Under File > Project Structure, add pdk module as a dependency for your Project

7. Add the following as a dependency in build.gradle file

```java
compile 'com.mcxiaoke.volley:library:1.0.+'
```

### Using Source code

1. Go to Android Studio | New Project | Minimum SDK

2. Select "API 11: Android 3.0" or higher and create your new project.

3. After you create a new project, go to File > New > New Module

4. Select "Import Existing Project as Module"

5. Enter the path to source code downloaded from this repo.

6. Add the following as a dependency in build.gradle file

```java
compile 'com.mcxiaoke.volley:library:1.0.+'
```
7. If the pdk module is not added as a dependency automatically, add it manually under File > Project Structure.

## Setting up your App 

### Registering Your App
Visit the [Pinterest Developer Site](https://dev.pinterest.com/docs/api/) and register your application. This will generate an appId for you which you will need in the next steps.

### Configuring PDKClient
Before you make any calls using the PDKClient in your app, you will need to configure it with your appId: 

Add the following lines in "onCreate" method of your Launch Activity

```java
//imports
import com.pinterest.android.pdk.PDKCallback;
import com.pinterest.android.pdk.PDKClient;
import com.pinterest.android.pdk.PDKException;
import com.pinterest.android.pdk.PDKResponse;


//inside onCreate
PDKClient.configureInstance(this, YOUR_APP_ID);
PDKClient.getInstance().onConnect(this);
```

Add the following line outside the application tag in the manifest file.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

And add the following Intent filter to your Login Activity

```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="pdk[YOUR_APP_ID]" />
</intent-filter>
```

If your APP ID is 1234, scheme would be "pdk1234"


## Example App

The PDK comes with a Example App. Just clone this repo and run to get started. 


## Getting Started

### Authenticating

Here, is sample code for authentication request.

```java
List scopes = new ArrayList<String>();
scopes.add(PDKClient.PDKCLIENT_PERMISSION_READ_PUBLIC);
scopes.add(PDKClient.PDKCLIENT_PERMISSION_WRITE_PUBLIC);

PDKClient.getInstance().login(this, scopes, new PDKCallback() {
    @Override
    public void onSuccess(PDKResponse response) {
    	Log.d(getClass().getName(), response.getData().toString());
        //user logged in, use response.getUser() to get PDKUser object
    }

    @Override
    public void onFailure(PDKException exception) {
        Log.e(getClass().getName(), exception.getDetailMessage());
    }
});
```

If the previous auth token isn't valid or this is the first time you've requested a token, this call will cause an app switch to either the Pinterest app or browser. To handle the switch back to your app, override `onActivityResult` in your Activity as follows:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    PDKClient.getInstance().onOauthResponse(requestCode, resultCode, data);
}
```

Learn more about scopes [here](https://dev.pinterest.com/docs/getting-started/authentication/).

### Making Requests
PDKClient has methods to make direct GET, POST, PUT and DELETE requests. The supported endpoints can be found on the [the Pinterest API](https://dev.pinterest.com/docs/api/) webpage. The methods signatures are:

```java
  /**
   *  Makes a GET API request
   *
   *  @param path         The path to endpoint
   *  @param params       Any parameters that need to be sent to the endpoint 
   *  @param callback     Instance of PDKCallback to receive success and failure callbacks
   */
   getPath(String path, HashMap<String, String> params, PDKCallback callback)
   
   
  /**
   *  Makes a POST API request
   *
   *  @param path         The path to the endpoint 
   *  @param params       Any parameters that need to be sent to the endpoint
   *  @param callback     Instance of PDKCallback to receive success and failure callbacks
   */
  postPath(String path, HashMap<String, String> params, PDKCallback callback) 
  
  
  /**
   *  Makes a PUT API request
   *
   *  @param path         The path to the endpoint 
   *  @param params       Any parameters that need to be sent to the endpoint
   *  @param callback     Instance of PDKCallback to receive success and failure callbacks
   */
  putPath(String path, HashMap<String, String> params, PDKCallback callback) 
  
  
  /**
   *  Makes a DELETE API request
   *
   *  @param path         The path to the endpoint 
   *  @param params       Any parameters that need to be sent to the endpoint
   *  @param callback     Instance of PDKCallback to receive success and failure callbacks
   */
  deletePath(String path, HashMap<String, String> params, PDKCallback callback) 
```

Using one of these methods is straightforward:

```java
  PDKClient.getInstance().getPath("me/", nil, new PDKCallback() {
      @Override
      public void onSuccess(PDKResponse response) 
      }
      
      @Override
      public void onFailure(PDKException exception) {
      }
  });
```

### PDKResponse and Pagination
You probably noticed that success callback on PDKCallback returns an object of type PDKResponse.

This object encapsulates any type of object that you could get back from a request. If the request is to get a user's boards, then you can call `response.getBoardList()` to get a List with objects of type `PDKBoard`. If the request returns a list of pins, call `response.getPinList()`.

If the response is paginated, you can use the response object to see if there is more data `response.hasNext()`. And request the data if it exists `response.loadNext(callback)`.

The only tricky part here is that you need to replace your most recent response object with the new one returned by the `response.loadNext(callback)` call. Otherwise calling load next on the old responseObject will load the same data again.


### Fields

The SDK has three basic resource types: [Users](https://dev.pinterest.com/docs/api/users/), [Pins](https://dev.pinterest.com/docs/api/pins/) and [Boards](https://dev.pinterest.com/docs/api/boards/). By default a request returns a select few of the fields available in each object. To request more fields you must specify the fields you want as a comma separated list in the request URL. For example, by default a pin request will return the fields url, note, link and id. Let's say that you only need id and note, but also want the pin's image. Your request would look something like:

```
https://api.pinterest.com/v1/me/pins/?access_token=<your-access-token>&fields=id,image,note
```

For a list of available fields on each object, refer to the bottom of the [Getting Started documentation](https://dev.pinterest.com/docs/getting-started/). 


## PDK Convenience API methods

PDKClient also comes with convenience methods for almost all of the PDK endpoints. In each of these calls you must provide a list of fields that you wish the endpoint to return. 

### Authenticated User (me) methods

```java

getMe(String fields, PDKCallback callback)

getMyPins(String fields, PDKCallback callback)

getMyBoards(String fields, PDKCallback callback)

getMyLikes(String fields, PDKCallback callback)

getMyFollowers(String fields, PDKCallback callback)

getMyFollowedUsers(String fields, PDKCallback callback)

getMyFollowedBoards(String fields, PDKCallback callback)

getMyFollowedInterests(String fields, PDKCallback callback)

```

### User methods

```java

getUser(String userId, String fields, PDKCallback callback)

```

### Board methods

```java

getBoard(String boardId, String fields, PDKCallback callback)

getBoardPins(String boardId, String fields, PDKCallback callback)

deleteBoard(String boardId, PDKCallback callback)

createBoard(String name, String desc, PDKCallback callback)

```

### Pin methods

```java

getPin(String pinId, String fields, PDKCallback callback)

deletePin(String pinId, PDKCallback callback)

createPin(String note, String boardId, String imageUrl, String link, PDKCallback callback)

```

## Related Articles
[Pinterest API Docs](https://dev.pinterest.com/docs/getting-started/)
[Pinterest API signup](https://api.pinterest.com/developers/signup/)
