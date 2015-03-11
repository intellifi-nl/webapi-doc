Brain web API documentation
===========================

This document contain the offical Brain web API documentation. We provide a RESTful API that allows you to access all the data that we provide in a powerfull and simple way.

At this moment we only support JSON as output format.

By default the api is accessible on: http://`host`/api/`resource`/`id`
* The `host` will be provided to you when you are evaluating or purchasing our product. We always have an 'play-arround' brain that we can supply to you.
* The `resource` shall will contain the resource that you want to query. This is most of the times the plural form of a noun.
* The **optional** `id` indicates which specific resource you wish to access. Please refer to the individual resources for more information on the type of id that is used. If you omit `id` the server will return a list with all items in the resource.

As with every web API you can only request new information by doing an extra request. We offer a whole set of [pushing technologies](https://github.com/intellifi-nl/doc-push). They will allow you to be informed when something changes, instead of polling for changes.

Contents
--------

* [Terminology](#terminology)
  * [Item](#item)
  * [Zone](#zone)
* [Resources](#resources)
  * [Overview](#overview)
* [Design](#design)
  * [Explorability](#explorability)
  * [Authentication](#authentication)
  * [Pagination](#pagination)
  * [Versioning](#versioning)
  * [Todo](#todo)

Terminology
===========

The whole Intellifi concept is based upon items and zones. Please take some time to familiarize yourself with these definitions. They will make it way easier to understand this API.

Item
----

An item is a small and lightweight electronic device that contains and remembers a unique code. An item must be able to transmit this unqiue code wireless. This item may be a passive RFID tag, but it can also be an iBeacon. Even your smartphone could behave itself as an item. A device is an item as long as it's able to remember and transmit it's unique code.

An item is an abstraction that allows us to work with RFID tags and iBeacon tags as if they where the same. Attributes are added to the items so that you can see the difference, if you wish.

Zone
----

A zone is an area in which items can be detected. This area is naturually limited to the range of the used RFID technology. Passive tags have a range of approximately 10 meters, active tags can easily have a range of 100+ meters. A zone must be able to reports it's detected items.

A lot of devices are capable of behaving themselves as 'zones'. Our [Intellif Spot](http://intellifi.nl/home/products/) is a very good example of this. It can detect RFID tags and Bluetooth tags that are in the neighbourhoud. It reports these detections through it's network interface to a server. It's also possible to connect external antennas to the Spot. By default they are used to enlarge the reach of the overall Spot zone. You may configure individual external antenna to behave themselve as a zone as well. By doing so you are essentially creating a virtual spot. 

Another great example of a zone could be your smartphone. Lot's of smartphones support the detection if iBeacons. It's a matter of the right app to report this information to a server. And voilà: here's another zone that you can use to detect your items.

So you see that a zone is an abstraction, it can be 'implemented' by multiple devices. Therefore it's not avaialble as a seperate resource in our web API. You will see that most resources mention the word, it's a key concept in our solution.

Resources
=========

Resources are the core of the web API. We tried to limit the number of resources to the core concepts. An elaborate description of each resource is available on [this page](resources.md).

Overview
--------

TODO: Add a table with all available resources.

Create and updating
-------------------

You can create and change most avaialble resources in the API. This can be done by sending an HTTP POST (for creating) or PUT (for updating). Your programming language will most likely have several options to achieve this. In this example we will use the wel known unix tool curl (also avaialble for windows). Many other tools are avaialble.

We will first create a new location by running post:
```
curl -XPOST -H "Content-Type: application/json" --data @postBody.json http://`host`/api/locations
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

More fields may be added in the future without further notice.

We now want to change the value of the label to a more sensible value. Please note that you will have to add the id of the resource to your uri.
```
curl -XPUT -H "Content-Type: application/json" --data @putBody.json localhost:3000/api/locations/550052669dba59d03057cb74
```

The contents of the file `putBody.json`:
```
{
  "label":"Kitchen"
}
```

This will result in a reply similar to the POST command. Now the label is changed to Kitchen.

Design
======

API design decisions are described in this chapter. It may answer some questions that you have.

Explorability
-------------

We find it very important that our web API is self explaining. We strongly recommand you to install a JSON viewer plugin in your webbrowser. This will allow you to view the query results in your web browser. For Google Chrome we advice you to use [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own personal browser as well.

Most of the resources include links to other relevant resources. These links are added as fields to the JSON objects. A good JSON viewer will allow you to follow them with a simple click. These fields always have the "url_" prefix.

Id
--

Every resource has an unique identiefer (ID). By default we embed this id into the first given field: `url`. You may add the `id_only` = true parameter to the query part of the url if you wish to have this id as a single field. The `url` field is then replaced by the `id` field. This also applies to references to other resources. You will probably want to do this in integrations only, for exploring the API it's way easier to just keep and use the `url` field.

Reference
---------

Most resource contain fields that reference to another resource in the API. An item resource will be pointing to some location i.e. This reference is very important an can be represented in three ways:

1. By default it's shown as an url. This helps you in navigating to it, if you have the right browser extension then you can just click it. The '_url' is postfixed to the field name. I.e. location becomes location_url. 
2. If you add `id_only=true` in the query then only the id of the resource is shwon. The name of the field is postfixed with '_id'. I.e. location becomes location_id
3. If you add `populate=fieldname,fieldname2,etc` in the query string then API will try to add the documents as an extra level in the results. The name of the field is not appended with a postfix in this case. If the lookup fails then `null` is returned as value.

Authentication
--------------

TODO: API keys, users etc.

Pagination
----------

The number of results is always limited to 100. Obviously we do allow you to make more querys so that you can retreive the rest of the results. This process is called paginiation and keeps our server load at acceptable levels.
* Default list/listing envelope
* Links that help you
* TODO: Implement RFC specific headers?
* TODO: Be carefull when resources are added or deleted during a paginiation. If you don't want to miss a thing then the events resource is the only reliable source.

Versioning
----------

TODO: /always the latest versions, /v2, /v1

Todos
-----

* CORS
* Explain time format and link to iso.
* https is not yet supported.
* expand for items themself, and their relations! By default only the items, you can request expands for seperate things?
* option for extra navigational url's?
* Time, UTC only on our side. Presentation layer can add a timezone.
* Move the resources to a seperate page and give only an overview on this page.
* The first returned JSON field is always an URL to the current resource or query, the second is always the id if you are working with a resource.
* url_spot
* id_spot
* This is nice, we could also choose to disable 
