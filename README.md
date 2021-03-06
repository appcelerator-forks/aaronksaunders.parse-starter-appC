[![Appcelerator Titanium](http://www-static.appcelerator.com/badges/titanium-git-badge-sq.png)](http://appcelerator.com/titanium/) [![Appcelerator Alloy](http://www-static.appcelerator.com/badges/alloy-git-badge-sq.png)](http://appcelerator.com/alloy/)
____
[![Building Cross-Platform Apps using Titanium, Alloy, and Appcelerator Cloud Services](images/Book Info.png)](http://www.clearlyinnovative.com/build-it/)
____
#Appcelerator Titanium Alloy Mobile - Parse Starter Template w/REST Service

This approach differs from the previous template in that we focus on using the [Parse REST API](https://www.parse.com/docs/rest) instead of the JavaScript API. The work is is [based on work done by Stephen Feather](https://gist.github.com/sfeather/4400387) and other on creating the initial gist that I have derived the parseREST service from. I have made additional updates to the parseREST service and will continue to make changes to the application to make for a better starter template as an alternative to Appcelerator Cloud Services

#### Misc Setup 
I use `gittio` for installing and maintaining modules and recommend that you do to. See [http://gitt.io/cli](http://gitt.io/cli) for additional information on using this great service.

######Android Push - Install Android Push Module
It can be downloaded from here, we only are using it for the Android Parse Push Connectivity
[https://github.com/timanrebel/Parse](https://github.com/timanrebel/Parse)


######Pull to Refresh - Install Module
[`gittio install nl.fokkezb.pullToRefresh`] ( http://gitt.io/component/nl.fokkezb.pullToRefresh)

For Android, you need to install and additional module
[`gittio install com.rkam.swiperefreshlayout`](http://gitt.io/component/com.rkam.swiperefreshlayout)

######The Application Keys for Parse
These keys need to be set in `tiapp.xml` the application can use the values passed in on the `parseService.init` call, but these values will not be accessible in the Android Push Module so it is recommended to use the `tiapp.xml` approach.
```
<property name="Parse_AppId">Parse_AppId</property>
<property name="Parse_ClientKey">Parse_ClientKey</property>
<property name="Parse_RESTAPIKey">Parse_RESTAPIKey</property>
```


Subscribe to the video channel [Beginning Appcelerator Titanium Alloy - Video Channel](https://www.youtube.com/channel/UCMCcqbJpyL3LAv3PJeYz2bg), for additional updates on this and other Appcelerator Services and Templates

###Working with the Service
Please note that this implementation does not require a sync adapter, the service returns javascript objects. If you want to use the data-binding capabilities of Appcelerator Titanium Alloy, you can always just create a Model or Collection with the results
````Javascript
// add library to project
var parseService = require('parseREST');

// initialize service, keys are set in tiapp.xml
parseService.init();
````
###Working with Users
Logging in a existing user; notice the integration of promises into the service to provide for a cleaner architecture
````Javascript
parseService.loginUser("aaronsaunders", "password").then(function(_result) {
    console.log(JSON.stringify(_result, null, 2));
}, function(_error){
    Ti.API.error('ERROR: ' + JSON.stringify(_error, null, 2));
});
````
If you have logged in previously, you can restore the user's session without logging in
````Javascript
parseService.restoreUser().then(function(_result) {
    console.log(JSON.stringify(_result, null, 2));
}, function(_error){
    Ti.API.error('ERROR: No session currently exists - ' + JSON.stringify(_error, null, 2));
});
````
###Working with Objects
[See objects below](#screenshot-of-tutor-sessions-objects); but querying the objects are pretty straight forward and url parameters that are documented in the [Parse REST API documentation](https://www.parse.com/docs/rest) can be passed in using the `urlparams` property on the `params` method variable
````Javascript
var _urlparams = {
    include : 'user,tutor,place', // return child objects for relationships
    limit : 5                     // only return the first 5 elements
};
var params = {
    'urlparams' : _urlparams
};
parseService.getObjects('TutorSession', params).then(function(_queryResult){
  _.each(_queryResult.response.results, function(element) {
    // log information about the tutoring session
    console.log("Tutor: " + element.tutor.first_name + " " + element.tutor.last_name);
    console.log("Location: " + element.place.Location + ", " + element.place.Name);
  });
}, function(_error){
    console.log("Some Error Happened: " + JSON.stringify(_error));
});
````
A more complex query using the `where` paramater can be done as follows:
``` Javascript
// find all tutoring sessions that are being done by
// the tutors with the specified ids
var whereQueryStr = {
    "tutor" : {
        "$inQuery" : {
            "where" : {
                "objectId" : {
                    "$in" : ["SCV3V0GqRr", "9uKbg0Hzeb"] //<== set ids here
                }
            },
            "className" : "_User" // <== specify the class of the pointer relation
        }
    }
};

return parseService.getObjects('TutorSession', {
    "urlparams" : {
        "where" : whereQueryStr
    }
}).then(function(_queryResult){
  _.each(_queryResult.response.results, function(element) {
    // log information about the tutoring session
    console.log("Tutor: " + element.tutor.first_name + " " + element.tutor.last_name);
    console.log("Location: " + element.place.Location + ", " + element.place.Name);
  });
}, function(_error){
    console.log("Some Error Happened: " + JSON.stringify(_error));
});
```

As mentioned above, if you want to work with Alloy Collections, you can just set the Collection with the results
``` Javascript
var collection  =  Alloy.createColllection('Sessions',_queryResult.response.results);
```
####Screenshot of Users Objects
[![Appcelerator Alloy](images/parse_users.png)](images/parse_users.png)

####Screenshot of Places Objects
[![Appcelerator Alloy](images/parse_places.png)](images/parse_places.png)

####Screenshot of Tutor Sessions Objects
[![Appcelerator Alloy](images/parse_tutor_sessions.png)](images/parse_tutor_sessions.png)

###Working with Files and Binary Objects
When working with Files in Parse, you have to remember that the only way to get access to the File object after it is created is to associate it with another object. In the example below, we will do this in two steps, first show how to create the file, and then show how to associate it with an object.

``` Javascript
var parseService = require('parseREST');
parseService.init();

file = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "iTunesConnect.png");
var blob = file.read();

parseService.uploadFile("image/jpeg", "iTunesConnect.png", blob);
```
This code will give you back a response object something like this and the file is floating out there in parse and there  is not real way to get the image back.

``` Javascript
{
    "name":"tfss-e74daf28-690a-4527-8c64-f99e5641b6e5-iTunesConnect.png",
    "url":"http://files.parsetfss.com/f875e1c9-4326-4040-afd8-19849220/tfss-e74daf28-iTunesConnect.png"
}
```
A helper method with allow you to upload the file and associate it with a specific object called a ImageInfo. This `ImageInfo` object will provide access to the image
```Javascript
var parseService = require('parseREST');
parseService.init();

file = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "iTunesConnect.png");
var blob = file.read();

parseService.uploadFile("image/jpeg", "iTunesConnect.png", blob).then(function(_results) {
	return parseService.createObject('ImageInfo', {
		"caption" : _results.response.name,
		"picture" : {
			"name" : _results.response.name,
			"__type" : "File"
		}
}).then(function(_results2) {
	console.log("FileHelper Object: " + JSON.stringify(_results2));
},function(_error)
    console.log("ERROR: " + JSON.stringify(_error));
});
```
The results should look something like this:
```
{
    "createdAt": "2015-05-11T15:30:52.004Z",
    "objectId": "yLPdeXDinq"
}
```
####Image Resizing
Most of todays newer phones create this amazing high resolution images, for this specific application I have chosen to utilize an Appcelerator module to resize the images by 50% before uploading them to the Parse server. This allow for a much better user experience when working with the images and waiting for them to upload and still you end up with a decent quality photo.

```Javascript
if (OS_ANDROID || _imageData.width > 700) {
    var w,
        h;
    w = _imageData.width * .50;
    h = _imageData.height * .50;
    imageCompressed = _imageData.imageAsResized(w, h);
} else {
    // we do not need to compress here
    imageCompressed = _imageData;
}
```
Additional information & documentation on the [`Ti.Blob.imageAsResized`](https://docs.appcelerator.com/platform/latest/#!/api/Titanium.Blob-method-imageAsResized) module can be found here  by clicking the link which will direct you to the Appcelerator Documentation.

We also us the Appcelerator blob methods to create a thumbnail image to store with the File object, we use this smaller image to render in the list view to provide better performance for the application. 
```Javascript
// create a small thumbnail for display purposes in base64 and save in row
// this way we don't need to render the huge original image
thumbBlob = imageCompressed.imageAsThumbnail(128);

// store as a base64 string
thumbBase64 = Titanium.Utils.base64encode(thumbBlob);
```
In Parse, we dont want to create another file object for the thumbnail so we convert it to base64 and save it as a string. See the change in how we save the object below.
```Javascript
parseService.uploadFile("image/jpeg", Ti.Platform.createUUID() + ".jpeg", imageCompressed).then(function(_results) {
    return parseService.createObject('ImageInfo', {
        "caption" : _results.response.name,
        "thumbBase64" : thumbBase64 + "", // <== SAVING THUMBNAIL
        "picture" : {
            "name" : _results.response.name,
            "__type" : "File"
        }
    }).then(function(_results2) {
        console.log("FileHelper Object: " + JSON.stringify(_results2));

        return $.photoListView.loadImages();
    }, function(_error) {
        console.log("ERROR: " + JSON.stringify(_error));
    });
});
```
When we need to render the thumbnail in the listview, we decode the base64 string.
```Javascript
Titanium.Utils.base64decode(_photos[i].thumbBase64 + "")
```
####Screenshot of PhotoList View Showing Uploading Images in Parse
[![Appcelerator Alloy](images/PhotoListView.png)](images/PhotoListView.png)
----------------------------------

Appcelerator, Appcelerator Titanium and associated marks and logos are 
trademarks of Appcelerator, Inc. 

Titanium is Copyright (c) 2008-2015 by Appcelerator, Inc. All Rights Reserved.

Titanium is licensed under the Apache Public License (Version 2). Please
see the LICENSE file for the full license.

