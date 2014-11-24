doc-webapi
==========

This document contain the offical documentation on the Intellifi brain web API.

Overview
========
We provide a RESTful API that allows you to access all the data that we provide in a powerfull and simple way.

At this moment we only support JSON as serialisation format.

By default the api is accessible on: http(s)://{host}/api/{resource}/{id}

The host will be provided to you when you are evaluating or purchasing our product. We always have an 'play-arround' brain that we can supply to you.

The {resource} shall will contain the resource that you want to query. This is most of the times the plural form of a noun.

The {id} indicates which specific resource you wish to access. Please refer to the individual resourcse for more information on the type of id that is used. In most resources this is a [MongoDB ObjectId](http://docs.mongodb.org/manual/reference/object-id/). You can omit the {id}: the server will return a list with all items in the resource (not if it's to much: see paginiation).

TODO: State vs events (seperate access to message bus and websocket)

Explorability
=============
We find it very important that our web API is self explaining. We strongly recommand you to install a JSON viewer plugin in your webbrowser. This will allow you to view the query results in your web browser. For Google Chrome we advice you to use [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own personal browser as well.

Most of the resources include links to other relevant resources. These links are added as fields to the JSON objects. A good JSON viewer (see previous paragrah) will allow you to follow them with a simple click. These fields always have the "url_" prefix.

Resources
=========
* items
* spots
* locations
* presences
* events
TODO: sets, brain senses

Pagination
==========
The number of results is always limited to 100. Obviously we do allow you to make more querys so that you can retreive the rest of the results. This process is called paginiation and keeps our server load at acceptable levels.
* Default list/listing envelope
* Links that help you
* TODO: Implement RFC specific headers?

Todo
====
* Authentication
* API keys
* Versioning
* CORS
