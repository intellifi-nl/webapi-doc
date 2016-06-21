Getting Started
=========================

This page describes some tools and basics that you need to know in order to get started with our web API. First of all you will need some knowledge of [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) and [JSON](https://en.wikipedia.org/wiki/JSON). They are the building blocks of our ecosystem (and many others of course).

Contents
--------

* [Exploring](#exploring)
* [Postman](#postman)
* [Curl](#curl)

Exploring
---------

We find it very important that our web API is self-explanatory. We strongly recommend you to install a JSON viewer plugin in your webbrowser. This will allow you to view (and navigate!) the query results in your web browser. For Google Chrome we advise you to use [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own favorite browser as well.

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore2.png)

Most of the offered API resources include links to other relevant resources. These links are added as fields to the JSON objects. A good JSON viewer will allow you to follow them with a simple click.

Postman
-------

Getting started with our API can be done quickly by downloading [Postman](https://www.getpostman.com/). It's an easy tool that allows you to create PUT and POST requests from within Chrome. You can exactly see what's sent and what's received. We prepared a set with example urls that allows you to experiment. It should look like this:
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-get.png)

First you should use the import function of postman to get the collection with example urls.

Copy this value to your clipboard: `https://www.getpostman.com/collections/34a4b3c71558d6115f72`

Now paste the url into the import function:
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-import.png)
Note: postman gets a lot of updates, so it might look a bit different. Just search for the import and you should be fine.

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-env.png)
You should also create and configure an environment. This allows you to tell postman the address of your brain and API key. 
Setup the environment:
- Select from the dropdown menu at the right side of the screen `Manage Environment` and click on it (see the figure above).
- Click on the `Add` button to create a new environment.
- Replace the text `New environment` with the name of your new environment.
- Configure the following key/value pairs:
  - `brain_host` to your brain server address (i.e. `brain.intellifi.nl`). Please make sure that you don't include the protocol name and the trailing slash!,
  - `api_key` to your API key.
- Click on `Submit` to save the configuration.
- Select the environment you just have created from the dropdown menu to make it active.

Tip: You can press the little arrow to get a very nice overview in which I added some extra comments to the urls.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/postman-overview.png)

Important note: some example url's include document ids, they should be replaced by resources that actually exist on your server. You can just copy and paste them from your browser.

Curl
----
A few basic examples using Curl.

1. Add a new location.
  The body contains the label for our new location.
  ```
  curl -X POST -H "Content-Type: application/json" -d '{"label": "My First Location"}' https://`brain_host`/api/locations
  ```

  Result:
  The reply to this POST will be another JSON object that contains information about the executed request. The most important is the url and the id of the newly created location resource:
  ```
  {
    "updated_properties_count":2,
    "url":"http://localhost:3000/api/locations/550052669dba59d03057cb74",
    "id":"550052669dba59d03057cb74",
  }
  ```
2. Get the new location resource.
  Use the ID returned in step 1.
  ```
  curl -X GET -H "Content-Type: application/json" https://`brain_host`/api/locations/550052669dba59d03057cb74
  ```
  
  Result:
  ```
  {
    "url": "https://'brain_host'/api/locations/550052669dba59d03057cb74",
    "id": "550052669dba59d03057cb74",
    "label": "My First Location",
    "custom": null,
    "time_created": "2016-06-15T13:33:23.399Z",
    "time_updated": "2016-06-15T13:33:23.399Z"
  }
  ```

3. Change the label of our new location.
  Now we want to change the label to a more sensible value. Please note that you will have to add the id of the resource to your url.

  ```
  curl -X PUT -H "Content-Type: application/json" -d '{"label": "Kitchen"} https://`brain_host`/api/locations/550052669dba59d03057cb74
  ```

  This will result in a reply similar to the POST command. Now the label is changed to 'Kitchen'.
