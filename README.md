Brain web API documentation
===========================

This document contain the official Brain web API documentation. The Brain web API is a [RESTful API](https://en.wikipedia.org/wiki/Representational_state_transfer) that allows you to access all the data that we provide in a powerful and simple way.

By default the api is accessible on: http://`brain_host`/api/`resource`/`id`
* The `brain_host` will be provided to you when you are evaluating or purchasing our product. We always have an 'play-arround' brain that we can supply to you.
* The `resource` shall contain the resource that you want to query. This is always the plural form of a noun. I.e. items, spots, events.
* The **optional** `id` indicates which specific resource you wish to access. Please refer to the individual resources for more information on the type of id that is used. If you omit `id` the server will return a list with all items in the resource.

The default data format is [JSON](https://en.wikipedia.org/wiki/JSON).

As with every web API you can only request new information by doing another request. We do offer a whole set of [pushing technologies](https://github.com/intellifi-nl/doc-push). They will allow you to be directly informed when something changes, instead of polling for the changes. Most use cases that we had until now can be implementated by using the web API only.

Contents
--------
* [Getting started](#getting-started)
* [Terminology](#terminology)
  * [Item](#item)
  * [Zone](#zone)
* [Resources](#resources)
  * [Overview](#overview)
  * [Id](#id)    
  * [References](#references)  
* [Future](#future)

Getting started
===============

We do advise you to take a quick look at our [quick start](quick-start.md) for some hints and tooling. In the rest of the documentation we assume that you understand HTTP, JSON and that you know which tools you can use to work with them.

Terminology
===========

The whole Intellifi concept is based upon items and zones. Please take some time to familiarize yourself with these definitions. They will make it way easier to understand this API.

Item
----

An item is a small and lightweight electronic device that contains and remembers a unique code. An item must be able to transmit this unqiue code wireless. This item may be a passive RFID tag, but it can also be an iBeacon. Even your smartphone could behave itself as an item. A device is an item as long as it's able to remember and transmit its unique code.

An item is an abstraction that allows us to work with RFID tags and iBeacon tags as if they where the same. Attributes are added to the items so that you can see the difference, if you wish.

Zone
----

A zone is an area in which items can be detected. This area is naturually limited to the range of the used RFID technology. Passive tags have a range of approximately 10 meters, active tags can easily have a range of 100+ meters. A zone must be able to reports its detected items.

A lot of devices are capable of behaving themselves as 'zones'. Our [Intellif Spot](http://intellifi.nl/home/products/) is a very good example of this. It can detect RFID tags and Bluetooth tags that are in the neighbourhoud. It reports these detections through its network interface to a server. It's also possible to connect external antennas to the Spot. By default they are used to enlarge the reach of the overall Spot zone. You may configure individual external antenna to behave themselve as a zone as well. By doing so you are essentially creating a virtual spot. 

Another great example of a zone could be your smartphone. Lots of smartphones support the detection of iBeacons. It's a matter of the right app to report this information to a server. And voilà: here's another zone that you can use to detect your items.

So you see that a zone is an abstraction, it can be 'implemented' by multiple devices. Therefore it's not available as a seperate resource in our web API. You will see that most resources mention the word, it's a key concept in our solution.

Resources
=========

Resources are the core of the web API. We tried to limit the number of resources to the core concepts. An elaborate description of each resource is available on [this page](resources.md).

Overview
--------

| Name | Description | 
| ----- | ---- | ----------- |
| [items](resources.md#items) | All detected items, now and in the past. |
| [locations](resources.md#locations) | The locations that devices may report to. |
| [spots](resources.md#spots) | Live information about your Intellifi Smart Spots. |
| [presences](resources.md#presences) | All the presences in your system. |
| sets | Combine items into lists. |
| services | Overview of running background processes on the brain. |
| [events](resources.md#events) | Access to all archived events that flowed through the brain. |

Id
--

The first two fields of every resource are `url` and `id`. They both uniquely identify the resource. You may add the `id_only=true` condition to the query part of the url if you wish to hide the `url` field(s). This will also apply to resource references. The `url` fields are shown by default to increase explorability of the API. If you are creating an automated import then you probably want to add `id_only=true` to all your requests.

References
----------

Most resources contain one or more fields that reference to another resource in the API. I.e. An item resource will be pointing to some location. These references can be shown in three ways. Depending on the arguments that you supply in the query part of your request url.

1. By default it's shown as a url. This helps you in navigating to it, if you have the right browser extension then you can just click it. The '_url' is postfixed to the field name. I.e. location becomes location_url.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore.png)

2. If you add `id_only=true` in the query then only the id of the resource is shown. The name of the field is postfixed with '_id'. I.e. location becomes location_id
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore_idonly.png)

3. If you add `populate=fieldname,fieldname2,etc` in the query string then the brain will try to add the documents as an extra level in the results. The name of the field is not appended with a postfix in this case. If the lookup fails then `null` is returned as value.
![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/explore_populate.png)

Future
------

A lot of things are going to happen in the future, some things need to be developed. And other things only need to be documented. Please let us know if you are really waiting for something.
* Authentication
* Versioning
* SSL
* Explain time format and link to iso (UTC only).
* Explain default CORS setting.
* Explain item sets
