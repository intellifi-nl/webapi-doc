doc-webapi
==========

This document contain the offical documentation on the Intellifi brain web API.

Overview
We provide a RESTful API that allows you to access all the data that we provide in a powerfull and simple way.

By default the api is accessible on: {brain-url}/api/{resource}/{id}

The {brain-url} will be provided to you when you are evaluating or purchasing our product.

The {resource} shall will contain the resource that you want to query. This is most of the times the plural form of a noun.

The {id} indicates which specific resource you wish to access. Please refer to the individual resourcse for more information on the type of id that is used. In most resources this is a MongoDB ObjectId (http://docs.mongodb.org/manual/reference/object-id/). You can omit the {id}, The server will respond with a list of items in the resource.

TODO: State vs events (seperate access to message bus and websocket)

Explorability
We find it very important that our web API is self explaining. We strongly recommand you to install a JSON viewer in your webbrowser. This will allow you to view the query results in your web browser. For Google Chrome we advice you to use JSONView (https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own personal browser as well.

Most of the resources include links to other resources. These links are added to the 
It will even allow you to follow links that we provide in most of the resources.
We do add  we are adding direct links to other resources that have something todo with the information that you requested. These links fields always start with "url_".

Resources
* items
* spots
* locations
* presences
* events
TODO: sets, brain senses

List
* Default list envelope
* Pagination

Todo
* Authentication
* API keys
* Versioning
* CORS
