---
title: Firebase Chat Client
subtitle: A Chat Client Using Firebase Cloud Messaging Service ( FCM)
hashtag: Google Cloud Messaging 
date: 2016-11-26 22:25:37
tags: html, web authentication, javascript
---

## Brief intrro to GCM 


Web technologies have been given alot of attention for obvious reasons and on of the reasons is the flexibility in connectivity with the users. Speaking of connectivity, this article is going to be demonstrating how to make use of Google's cloud messaging service to build a chat client. You might be wondering why am talking about Google Cloud Messaging service the answer is that they are both the same. Although gcm is depreciated, __fcm__ still uses google cloud messaging service. . Al be referencing the service as gcm from now on because we are not going to connect it with firebase databases.



### Why should i use it ?



GCM is not a messaging service, behind the scene your application establishes secure socket connection with google's connection server and tranports data through those connections. This means that it allows for parallel data transmission which means that you can make a broadcast which is big avantage if you need to make every person subscripbed to a chat room recieve a message at the same time. GCM is a google's cloud hosted service which means that the operations are very fast and the latency reliable which maybe an obvious point. Here are instances when you may use GCM .


1 . Sending messages from app to another app
2 . Sending messages from app to server 
3 . Sending messages from server to app




### When not to use it 



 Even though gcm has good perfomance there are times it might limit your creativity, one thing become an issue when your actually making a chat client or application and not just connecting applications together. Storage, i was using localStorage but i came short because localStorage has a limit of 2Mb per origin and chat clients involve heavy data transmission sometimes even media data like audio, video, pictures and music files which is not also allowed. Also GCM allows only 4kb payload per transmission, if your looking to make a chat client that involves media content like images and videos you might want to look at other alternatives like the Firebase Messaging Service [FCM](https://firebase.google.com/docs/cloud-messaging/). You don't have to deal with databases in GCM you can store messages in the sandbox storage, again there is a limit of 5mb per origin in chrome.





## Generating app credential


 Credential identifies the owner of the app and also allows you to pass through authentication and use the service. You have to register first as google developer and access to the developer console. Use the steps below to generate your app credentials.



**Developer Console Login #1**


Login to cloud.google.com/console using your google account to login and make sure you on the right website below.

![img alt](http://localhost:50071/img/post_img/google-sigin.png)

You will be redirected to the developer console dashboard. Click on the top left menu icon and select the api manager.

![img alt](http://localhost:50071/img/post_img/google-developer-console.png)



**API selection #2**


click on Enable api icon at the top center. You will be redirected in a page with list of google apis, select google clound messaging. You will be redirected to the firebase cloud messaging website.


![img alt](http://localhost:50071/img/post_img/google-api-list.png)

            
**Firebase Developer Console #3**

 Click on web setup and you will find some documentation explaining about the implementing a firebase cloud messaging client on chrome. Create a new firebase project or link an exsisting GCM project to firebase. After creating a new project you will be redirected to the project managment firebase console .




**Firebase Project Creation #4**


In this page you get the option of adding firebase to your app , click on the add to wesite icon to get the credentials.

![img alt](http://localhost:50071/img/post_img/firebase-console-project-creation.png)

We are going to be using this credentials to make pass through to the api, this application is manyly for demonstration purpose otherwise the way am handling the credentials is not recomended in a real application.

![img alt](http://localhost:50071/img/post_img/firebase-console-credentials-sample.png)


Notice that you do not have a database yet, you have just created a cloud messaging service. If require a database for your app you can go to the database tab and start a new instance.

![img alt](http://localhost:50071/img/post_img/firebase-database-image.png)




### Advantages of the Firebase Cloud messaging service.




1 . Test hosting for partial deploys
2 . You get a database for your application
3 . rich notifications
4 . Analysis data for your app monitization
5 . Remote configuration
5 . Ease of social media authentication including email, twitter, facebook, github, Anonymous.




## Making the Chat client 


I was writting a different application when i encountered __GCM__  and i thought it was awesome. I have looked for a reason to use it and here i am helping someone else have an easy time using the __GCM__ api.


**App permisions and manifestation**


Using __GCM__ is easy and creating a chrome extensions is much easier but the manifest is important and more importantly is the permissiosn scope that is defines what your application is able to do. The manifest is just a json file with key pair values of app/extensions properties.You can read more about creating an app/extensions and app/extensions manifestation. Your application may run into issues where the application just silently fails and this be caused by lack of permissions. You have to include the GCM into the permissions scope of your mainfest. If you want to look at some other permissions you can include in your application here is a list permissions from the google developer [website]("https://developer.chrome.com/extensions/declare_permissions"). package.json file.




``` json
 {
 "name": "App name",
 "description": "Brief Application description ",
 "manifest_version": 2,
 "version": "0.1",
     "app": {
     "background": {
     "scripts": ["background_script.js"]
     }
 },
 "permissions": ["app","permisions","scope"],
 "icons": { "128": "applogo.png" }
 }
```





First let us create a html page that we are going to use to interact with.




``` html
 <!DOCTYPE html>
 <html>
 <head>
     <title>CalFit</title>
     <link rel="stylesheet" href="assets/bootstrap.min.css">
     <link rel="stylesheet" href="assets/login.css">
     <script src="assets/jquery.min.js"></script>
     <script src="register.js"></script>
 </head>
 <body>
     <div class="container">
         <div class="row">
             <h2 align="center">Register</h2>
         </div>
         <form id="register-form" action=" method="post" role="form">
             <div class="form-group">
                 <input type="text" name="name" id="name" tabindex="1" class="form-control" placeholder="Name" value=">
             </div>
             <div class="form-group">
                 <input type="email" name="email" id="email" tabindex="1" class="form-control" placeholder="Email Address" >
             </div>
             <div class="form-group">
                 <div class="row">
                     <div class="col-sm-6 col-sm-offset-3">
                         <input type="submit" name="register-submit" id="submit" tabindex="4" class="form-control btn btn-register" value="Register Now">
                     </div>
                 </div>
             </div>
         </form>
         <div class="alert alert-success" id="success" style="display: none;">
             <strong>Success!</strong>
         </div>
     </div>
 </body>
</html>

```







Background tasks are important because we expect to recieve messages even if the user is not active using the app. Add the background permissions your manifest. Code to be executed in the background is going to be inside the background.js file or any other that is in the same array.


``` javascript 
var isRegistered = true;
 function onIncomingMessages(msgDet) {
    // create notification 
    chrome.notifications.create(getNotificationId(), {
        title: 'GCM Message',
        iconUrl: 'notify.png',
        type: 'basic',
        message: msgDet
         }, function(error) {
        // throw error to the user
         console.error(error);
    });
 }
 // Make the first time installation users to register the application.
 function promptRegistration(argument) {
    if(!isRegistered){
    // create registration popup
     chrome.app.window.create(
      "main.html",
      {  width: 500,
         height: 400,
         frame: 'chrome'
      },
      function(appWin) {}
    );
    }else {
        // Already installed
    }
 }
// set listeners
chrome.gcm.onMessage.addListener(messageReceived);
chrome.runtime.onInstalled.addListener(onIncomingMessages);
chrome.runtime.onStartup.addListener(promptRegistration);

```


Before we proceed any futher for google to be able to uniqly identify each service subscriber, there has to be a key identifier that must not repeat. This key identifier is the chrome extension id to the users local machine but after registration as we are going to see is going to be the registration ID returned from google. That means you can choose any of the two be your key identity of an application depending on whatever your need be. In this case we need to identify the application from an external network view. We'll go with the regiration ID. Calling the registration method requires you to know the chrome extension ID available through the runtime.*api



``` javascript 
  /*
  main.js file
 */
function getClient() {
  if (chrome.runtime)
  return chrome.runtime.id;
}
// lets trigger the registation method
var client = getClient();
chrome.gcm.register(client, function (_regId) {
    console.log('chrome gcm registration started .');
    if (chrome.runtime.lastError) {
        console.log("Registration failed: " + chrome.runtime.lastError.message);
    }
                                            
    chrome.storage.local.set({'redID' : _regId});
});

```





We are first getting the client identity and then regirstring the app/extension to the gcm services using the chrome extensions id.



            
## Main Application Functionlity 
            

In this section we are going to take a deeper look at the code functionality of the application .


### Ways of passing a message


    1 . Upstreaming messages
    2 . Downstreaming messages



GCM allows you to downstream and upstream a message, where upstreaming is passing a message from a server to a client application. And downstreaming means recieving messages from a client. Both operations dont have a clear meaning it depends on the senariol. Upstreaming messages doesn't require background task except checking for connectivity to the internet. The Downstream operation requires us to wait for incomming and process the messges.

The image above shows basic working consept of the application, now lets take a look at sending messages and displaying them on the target client. We are going to put them inside of the main.js file. In this file we are going to need two functions one do display the incoming messages and the other to send the messages to subscripted clients.







``` javascript 
/*
  main.js file
*/
  // first lets get the chrome local id
function getClient() {
  if (chrome.runtime)
  return chrome.runtime.id;
}
  // lets trigger the registation method
  var client = getClient();
chrome.gcm.register(client, function (_regId) {
    console.log('chrome gcm registration started .');
    if (chrome.runtime.lastError) {
        console.log("Registration failed: " + chrome.runtime.lastError.message);
    }
                                                                        
  chrome.storage.local.set({'redID' : _regId});
});
//&#64;param {String} - Message text to be displayed to the client
// &#64;param {String} - Alphanumeric string to identify the destination client                                                                                               
 function onOutGoingMessages(msgDet,regID) {
    var msgInput = document.getElementById("msgInput");
    var message = msgInput[0].value;
    if(typeof(message)==="string"){
            var data = "registration_id=TARGET regID<your_regId>&data.greetings=message";
            // note the data object must be used
            var xhr = new XMLHttpRequest();
            xhr.withCredentials = true;
            xhr.addEventListener("readystatechange", function () {
            if (this.readyState === 4) {
                console.log(this.responseText);
            }
            });
            xhr.open("POST", "http://android.googleapis.com/gcm/send");
            xhr.setRequestHeader("authorization", "key=<your_apiKey>");
            xhr.setRequestHeader("content-type", "application/x-www-form-urlencoded");
            xhr.send(data);                        
        }
 }
  onOutGoingMessages();
```



## Handling uninstalled applications


For uninstalled application we can listen for uninstallation request from the user then unregister the application from the gcm service. We dont have to worry about the application being online the browser handles the rest for us.



``` javascript 
    //background.js file 
    var isRegistered = true;
function onIncomingMessages(msgDet) {
        // create notification 
        chrome.notifications.create(getNotificationId(), {
        title: 'GCM Message',
        iconUrl: 'notify.png',
        type: 'basic',
        message: msgDet
    }, function(error) {
        // throw error to the user
    });
}
    // Make the first time installation users to register the application.
function promptRegistration(argument) {
        if(!isRegistered){
        // create registration popup
        chrome.app.window.create(
        "main.html",
        {  width: 500,
            height: 400,
            frame: 'chrome'
        },
        function(appWin) {}
        );
        }else {
            // Already installed
        }
}
    // Add managment to your manifest
function onUninstalled (){
        chrome.management.onuninstalled = function (){
        chrome.storage.local.remove("regId");           
    }
}
// set listeners
chrome.gcm.onMessage.addListener(messageReceived);
chrome.runtime.onInstalled.addListener(onIncomingMessages);
chrome.runtime.onStartup.addListener(promptRegistration);
```



