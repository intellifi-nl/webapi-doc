Brain web API quick start
=========================

This page describes some tools and basics that you need to get started with our web API. First off all you will need some knowledge of [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) and [JSON](https://en.wikipedia.org/wiki/JSON). They are the building blocks of our eco system (and many others off course).

Exploring
---------

We find it very important that our web API is self explaining. We strongly recommand you to install a JSON viewer plugin in your webbrowser. This will allow you to view (and navigate!) the query results in your web browser. For Google Chrome we advice you to use [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own personal browser as well.

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore2.png)

Most of the offered API resources include links to other relevant resources. These links are added as fields to the JSON objects. A good JSON viewer will allow you to follow them with a simple click.

Postman
-------

Getting started with our API can be done quickly with [Postman](https://www.getpostman.com/). It's an easy tool that allows you to create PUT and POST requests from within Chrome. You can exactly see what's send and what's received. We prepared a set with example url's that allow you to experiment. It should look like this:
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-get.png)

First you should use the import function of postman to get the collection with example urls.

Copy this value: `https://www.getpostman.com/collections/45fa9260f58619764111`

Now just follow these steps:
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-import.png)
Note: postman gets a lot of updates, so it might look a bit different. Just search for the import and you should be fine.

You should also create and configure an environment. This allows you to tell postman the adress of your brain.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-env.png)

This environment screen allows you to configure key/value pairs. You only need to define the key `brain_host` to your brain server address, i.e. `brain.intellifi.nl`. Please make sure that you don't include the protocol name and the trailing slash!

Tip: You can press the little arrow to get a very nice overview in which I added some extra comments to the urls.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-overview.png)

Important note: some example url's include document ids, they should be replaced by resources that actually exist on your server. You can just copy and past them from your browser.

Curl
----

Off course you can also perform all given examples in curl, the good old unix tool for doing HTTP requests. You could also install curl on Windows if you like.

Adding a new location i.e. can be done by:
```
curl -XPOST -H "Content-Type: application/json" --data @`postBody.json` http://`brain_host`/api/locations
```

The body will be filled with the contents of the `postBody.json` file:
```
{
  "key":"value",
  "label":"My first! Yeah"
}
```

Please note that you may add multiple key/value pairs in the object. Unknown values will be ignored.

The reply to this POST will be another JSON object that contains information about the executed request. The most imporant is the url and the id of the newly created resource:
```
{
  "updated_properties_count":2,
  "url":"http://localhost:3000/api/locations/550052669dba59d03057cb74",
  "id":"550052669dba59d03057cb74",
}
```

We now want to change the value of the label to a more sensible value. Please note that you will have to add the id of the resource to your uri.
```
curl -XPUT -H "Content-Type: application/json" --data @`putBody.json` http://`brain_host`/api/locations/550052669dba59d03057cb74
```

The contents of the file `putBody.json`:
```
{
  "label":"Kitchen"
}
```

This will result in a reply similar to the POST command. Now the label is changed to Kitchen.
