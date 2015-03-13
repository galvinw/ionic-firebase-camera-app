In my opinion, [Firebase](http://www.firebase.com) and [Ionic Framework](http://www.ionicframework.com) are two of the coolest things around at the moment when it comes to mobile app development with cloud service integration.

What if we wanted to create an image service like Instagram, Google Picasa, or Imgur where users could take pictures and save them to the cloud for later?

In this particular tutorial we are going to see how to take pictures on a mobile device and bind them immediately to Firebase to be seen everywhere we are signed in.  In case you didn't know, images can be saved to Firebase instead of only text.

## Creating a New Ionic Framework Project

### The Prerequisites

We're going to need a few more things than [my other Firebase tutorial](https://www.airpair.com/ionic-framework/posts/ionic-firebase-password-manager) if you've been keeping up with my posts.  The list of our requirements is as follows:

* The latest [Firebase JavaScript](https://cdn.firebase.com/js/client/2.2.2/firebase.js) library
* The latest [AngularFire AngularJS](https://cdn.firebase.com/libs/angularfire/1.0.0/angularfire.min.js) library
* The AngularJS extension set for Ionic Framework, [ngCordova](http://ngcordova.com/)
* The [Apache Cordova Camera](https://github.com/apache/cordova-plugin-camera/blob/master/doc/index.md) plugin
* A Mac with the latest Xcode installed if building for iOS
* A device or simulator that supports a camera (iOS simulator unsupported)
* NPM, [Apache Cordova](http://cordova.apache.org/), Ionic, and Android installed and configured

### Creating Our Project

To start things off, we're going to create a fresh Ionic project using our Terminal (Mac and Linux) or Command Prompt (Windows):

```bash
ionic start ImageApp blank
cd ImageApp
ionic platform add android
ionic platform add ios
```

Note, if you're not using a Mac with Xcode installed, you cannot add and build for the iOS platform.

At this point we're left with a new Android and iOS project using the Ionic Framework **blank** template.

### Adding Our Plugins

This project will be using one external plugin that is supported by Apache Cordova.  The [camera plugin](https://github.com/apache/cordova-plugin-camera/blob/master/doc/index.md) will allow us to access the device or simulator camera for taking pictures.

With your Ionic project as the current working directory in your Terminal or Command Prompt, run the following:

```bash
cordova plugin add org.apache.cordova.camera
```

This will install the plugin for both Android and iOS.  For more information on seeing this plugin in action, you can visit one of [my previous tutorials](https://blog.nraboy.com/2014/09/use-android-ios-camera-ionic-framework/) on the topic.

### Adding Our Libraries

In our Ionic project's **www/index.html** we need to add all the various prerequisite libraries. The file's <head> should look something like the following:

```markup,linenums=true
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title></title>
    <link href="lib/ionic/css/ionic.css" rel="stylesheet">
    <link href="css/style.css" rel="stylesheet">
    <script src="lib/ionic/js/ionic.bundle.js"></script>
    <script src="js/ng-cordova.min.js"></script>
    <script src="cordova.js"></script>
    <script src="js/firebase.js"></script>
    <script src="js/angularfire.min.js"></script>
    <script src="js/app.js"></script>
</head>
```

It is very important to note that `ng-cordova.min.js` must be placed above `cordova.js`.  If you don't do this, you're going to get strange results.

## Preparing Our JavaScript File

For simplicity, we are going to be doing all custom JavaScript coding in our **www/js/app.js** file. It will contain all our controllers, factories, and prototypes.

### Naming Our AngularJS Module

For cleanliness, we are going to give our AngularJS module a name.

```javascript
var imageApp = angular.module("starter", ["ionic", "ngCordova"]);
```

In this case we've named our module **imageApp** and it will be used for all controllers and factories in our application.  Also take note that we've added `ngCordova` to the list.

### Getting Firebase Started

We're not quite ready to talk about Firebase, but we are at a point where it would be a good idea to initialize it. At the top of your **www/js/app.js** file, outside any AngularJS code, you want to add the following line:

```javascript
var fb = new Firebase("https://INSTANCE_ID_HERE.firebaseio.com/");
```

Of course you need to replace **INSTANCE_ID_HERE** with your personal Firebase instance. By setting it globally, it will load before AngularJS and can be used throughout your application.

We also need to add `firebase` to our AngularJS module. In the end it will look something like this:

```javascript
var imageApp = angular.module("starter", ["ionic", "ngCordova", "firebase"]);
```

## Using The AngularJS UI-Router

Our application will have the following two screens for simplicity:

* A screen for registering or signing into Firebase
* A screen for showing uploaded pictures and allowing users to upload new pictures

To accomplish different screens or views in an AngularJS application we will be using the UI-Router since it is already bundled with Ionic Framework. If you're familiar with my other tutorials, you might have already seen a [previous demonstration](https://blog.nraboy.com/2014/11/using-ui-router-navigate-ionicframework/) that I've done.

### Configuring Our States

With the AngularJS UI-Router, each view or screen is considered a state. In order to use we must configure the routes and the controllers associated with them. In your Ionic project's **www/js/app.js** file, let's add the following chunk of code which will represent the states for each of our screens:

```javascript,linenums=true
imageApp.config(function($stateProvider, $urlRouterProvider) {
    $stateProvider
        .state("firebase", {
            url: "/firebase",
            templateUrl: "templates/firebase.html",
            controller: "FirebaseController",
            cache: false
        })
        .state("secure", {
            url: "/secure",
            templateUrl: "templates/secure.html",
            controller: "SecureController"
        });
    $urlRouterProvider.otherwise('/firebase');
});
```

For simplicity, every time we open our app, we will be sent to the Firebase view forcing us to sign in.  If you want to get creative, you can check the authorization state and plan appropriately.

### Creating Our Views

We've just configured all of the routing information for our states so it is now time to set up our view templates. Essentially these are the HTML pages that will represent each of our screens.

The first step is to prepare our Ionic project's **www/index.html** file to use UI states. This is easy and can be accomplished in just a few lines. Navigate to the `<ion-pane>` lines and replace them with the following:

```markup,linenums=true
<ion-pane>
    <ion-nav-bar class="bar-stable"></ion-nav-bar>
    <ion-nav-view></ion-nav-view>
</ion-pane>
```

Time to go through the process of designing each of our views. Brace yourself. Our templates are going to be very simplistic. Feel free to be fancier in your view design.

Create and open the file called **www/templates/firebase.html** and add the following code:

```markup,linenums=true
<ion-view title="Firebase Login">
    <ion-content>
        <div>
            <div class="list list-inset">
                <label class="item item-input">
                    <input ng-model="username" type="text" placeholder="Username" />
                </label>
                <label class="item item-input">
                    <input ng-model="password" type="password" placeholder="Password" />
                </label>
            </div>
            <div class="padding-left padding-right">
                <div class="button-bar">
                    <a class="button" ng-click="login(username, password)">Login</a>
                    <a class="button" ng-click="register(username, password)">Register</a>
                </div>
            </div>
        </div>
    </ion-content>
</ion-view>
```

Now create and open the file called **www/templates/secure.html** and add the following code:

```markup,linenums=true
<ion-view title="My Images" ng-init="">
    <ion-nav-buttons side="right">
        <button class="button button-icon icon ion-camera" ng-click="upload()"></button>
    </ion-nav-buttons>
    <ion-content>
        <div class="row" ng-repeat="image in images" ng-if="$index % 4 === 0">
            <div class="col col-25" ng-if="$index < images.length">
                <img ng-src="data:image/jpeg;base64,{{images[$index].image}}" width="100%" />
            </div>
            <div class="col col-25" ng-if="$index + 1 < images.length">
                <img ng-src="data:image/jpeg;base64,{{images[$index + 1].image}}" width="100%" />
            </div>
            <div class="col col-25" ng-if="$index + 2 < images.length">
                <img ng-src="data:image/jpeg;base64,{{images[$index + 2].image}}" width="100%" />
            </div>
            <div class="col col-25" ng-if="$index + 3 < images.length">
                <img ng-src="data:image/jpeg;base64,{{images[$index + 3].image}}" width="100%" />
            </div>
        </div>
    </ion-content>
</ion-view>
```

At first glance the code we have in our **www/templates/secure.html** file may look nuts.  It really isn't.  It is a grid system that is better explained in a [previous article](https://blog.nraboy.com/2015/03/make-a-gallery-like-image-grid-using-ionic-framework) that I wrote.

## Configuring Our Firebase Instance

### Defining Permissions

Like my previous Firebase tutorials regarding building a [Firebase Todo List](https://blog.nraboy.com/2014/12/syncing-data-firebase-using-ionic-framework/) or [Creating a Password Manager](https://www.airpair.com/ionic-framework/posts/ionic-firebase-password-manager), we're going to be using the same permission strategy:

```json
{
    "rules": {
        "users": {
            ".write": true,
            "$uid": {
                ".read": "auth != null && auth.uid == $uid"
            }
        }
    }
}
```

Everyone will be able to write to the users node (create a new account), but only authorized users will be able to read data. In this case data will be the users own images. You can paste the JSON rules in the **Security & Rules** section of the Firebase dashboard.

### Allowing For Account Creation

In the **Login & Auth** section of the Firebase dashboard, we must enable **Email & Password** authentication. There are other types of authentication, but for this particular tutorial we're going to focus on email and password based.

Enabling this will allow people to register new accounts and create a unique `simplelogin:x` user key in our NoSQL data structure where `x` is an auto incrementing numeric value.

### Understanding The Data Structure

The data stored will be of very strict formatting. It will of course be JSON, but it will look like the following:

```json
{
    "images": {
        "unique_image_id_here": {
            "image": ""
        }
    }
}
```

When storing our images, Firebase will automatically create our unique image ids.  You'll see that shortly.

## Making Controllers To Pair With Our View Logic

### The Firebase Controller

The logic behind the `FirebaseController` is as follows:

* If the user chooses to sign in, check the email and password and redirect to the secure page if successful
* If the user chooses to register, create a new account and then immediately sign into and redirect to the secure page

```javascript,linenums=true
imageApp.controller("FirebaseController", function($scope, $state, $firebaseAuth) {

    var fbAuth = $firebaseAuth(fb);

    $scope.login = function(username, password) {
        fbAuth.$authWithPassword({
            email: username,
            password: password
        }).then(function(authData) {
            $state.go("secure");
        }).catch(function(error) {
            console.error("ERROR: " + error);
        });
    }

    $scope.register = function(username, password) {
        fbAuth.$createUser({email: username, password: password}).then(function(userData) {
            return fbAuth.$authWithPassword({
                email: username,
                password: password
            });
        }).then(function(authData) {
            $state.go("secure");
        }).catch(function(error) {
            console.error("ERROR: " + error);
        });
    }

});
```

Firebase stores sign in information on the device so that you can later make use of the `getAuth()` functions to see if you're currently authenticated.

### The Secure Controller

The logic behind the `SecureController` can be a little more complicated and is as follows:

* Clear the history stack so the back button doesn't bring us to the sign in screen again
* Create an empty array of images just in case no images exist on Firebase.  Don't want any undefined errors
* Check to see if the user is signed in
    * If not signed in, redirect back to the Firebase view
    * If signed in, pull down the array of images from the server
* If the user chooses to upload an image, open the camera
    * If the user accepts the camera image, then immediately push it to Firebase

```javascript,linenums=true
imageApp.controller("SecureController", function($scope, $ionicHistory, $firebaseArray, $cordovaCamera) {

    $ionicHistory.clearHistory();

    $scope.images = [];

    var fbAuth = fb.getAuth();
    if(fbAuth) {
        var userReference = fb.child("users/" + fbAuth.uid);
        var syncArray = $firebaseArray(userReference.child("images"));
        $scope.images = syncArray;
    } else {
        $state.go("firebase");
    }

    $scope.upload = function() {
        var options = {
            quality : 75,
            destinationType : Camera.DestinationType.DATA_URL,
            sourceType : Camera.PictureSourceType.CAMERA,
            allowEdit : true,
            encodingType: Camera.EncodingType.JPEG,
            popoverOptions: CameraPopoverOptions,
            targetWidth: 500,
            targetHeight: 500,
            saveToPhotoAlbum: false
        };
        $cordovaCamera.getPicture(options).then(function(imageData) {
            syncArray.$add({image: imageData}).then(function() {
                alert("Image has been uploaded");
            });
        }, function(error) {
            console.error(error);
        });
    }

});
```

Notice the `Camera.DestinationType.DATA_URL`.  This means that we are going to store the image as base 64 data in Firebase.

## Properly Testing Our App

### Using the Web Browser

Don't do it.  This app makes use of native device plugins that are loosely compatible with the web browser.  You're setting yourself up for failure if you even attempt to open this application in a web browser.

### Using a Device or Simulator

To test this application on your device or simulator, run the following in your Command Prompt or Terminal:

```
ionic build android
adb install -r platforms/android/ant-build/CordovaApp-debug.apk
```

The above will get you going on Android. If you wish to track down errors and view the logs, you can use the Android Debug Bridge (ADB) like so:

```
adb logcat
```

More information on this can be seen in one of my [other tutorials](https://blog.nraboy.com/2014/12/debugging-android-source-code-adb/) on the topic.

## Conclusion

You've just made your own little photo repository.  Is it the next Instagram?  Not in its current state, but you can definitely expand upon the material described in this article.

We took the base 64 image data obtained from the device camera and immediately pushed it into our Firebase storage.  This storage can be synchronized between all of your devices and platforms.
