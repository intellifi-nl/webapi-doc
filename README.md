Brain web API documentation
===========================

This document contain the official Intellifi Brain web API documentation. The Brain web API is a [RESTful API](https://en.wikipedia.org/wiki/Representational_state_transfer) that allows you to interact with our equipment in a powerful and simple way. Our end-to-end [solution](http://intellifi.nl/) allows you to localize your real world items based different RFID technologies. The results of these physicial detections are immediately available on our cloud based API (on-prem is available).

By default the API is accessible on: https://`brain_host`/api/`resource`/`id`
* The `brain_host` will be provided to you when you are evaluating or purchasing our product. Please contact us if you want to perform some experiments, we always have a play around brain available.
* The `resource` shall contain the resource that you want to query. This is always the plural form of a noun. I.e. items, spots, events.
* The **optional** `id` indicates which specific resource you wish to access. Please refer to the individual resources for more information on the type of id that is used. If you omit `id` the server will return a list with all items in the resource.

The default serialisation format is [JSON](https://en.wikipedia.org/wiki/JSON).

As with every web API you can request new information by performing HTTP requests. We do offer [pushing technologies](https://github.com/intellifi-nl/doc-push). They will allow you to be directly informed when something changes, instead of polling for the changes. Most use cases that we had until now can be implementated by using the web API only.

Contents
--------
* [Getting started](#getting-started)
* [Terminology](#terminology)
  * [Item](#item)
  * [Zone](#zone)
* [Resources](#resources)
  * [Overview](#overview)
  * [Collections](#collections)
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

An item is an object (or even a person) that you tagged with some kind of RFID emitter. At this moment our eco system can detect both RFID EPC Gen2 tags, also know as RAIN RFID tags, and modern Bluetooth LE beacons (including iBeacon and Eddystone). When an item is detected for the first time by one of our detectors then it's immediately created as an item resource. Our system keeps this resource as a reference to this item, it's never deleted. You can use it to see where the item is or where it has been seen for the last time.

Location
--------

A location is an area in which items can be detected.  The actual detections are made and transmitted to our server by devices ([Intellifi SmartSpot](https://intellifi.nl/home/products/)) that you install at the physical location. The events are transferred to the server by a default network connection.

One or more [Intellif SmartSpots](https://intellifi.nl/home/products/) allow the location to actually detect items. The detection area is naturally limited to the range of the used RFID technology. Passive EPC Gen2 tags have a range of approximately 10 meters, beacons can easily have a range of 100+ meters. If multiple SmartSpots are reporting to a one location then the events are merged at the server level. Please note that the location is a server-side abstraction that allows you to be flexible when you have multiple SmartSpots. I.e.: it's possible to connect external antennas to the SmartSpot. By default they are used to enlarge the reach of the overall SmartSpot. You may configure individual external antennas to report their detections to a seperate location. By doing so you are essentially creating a second virtual spot.

Location configuration is done automatically when you connect a SmartSpot to a brain for the first time. A location is created that caries the label 'spot`spot serial_number`'. I.e 'spot203'. You can easily change this label by doing a `PUT` on the location resource itself. We encourage you to add a meaningful name to a location (i.e. 'kitchen') as it's beeing used in user interface a several places.

Resources
=========

Resources are the core of the web API. We tried to limit the number of resources to the core concepts. An elaborate description of each resource is available on [this page](resources.md).

Overview
--------

| Name | Description | 
| ----- | ---- | ----------- |
| [items](resources.md#items) | All detected items, now and in the past. |
| [locations](resources.md#locations) | The locations that SmartSpots may report to. |
| [spots](resources.md#spots) | Live information about your Intellifi SmartSpots. |
| [presences](resources.md#presences) | All the presences in your system. |
| sets | Combine items into lists. |
| services | Overview of running background processes on the brain. |
| [events](resources.md#events) | Access to all archived events that flowed through the brain. |


Collections
-----------

The top level resources are all collections, they are wrapped into an object with information about the query. The collections are [paginated](pagination.md) when they contain more than 100 resources.

Id
--

The first two fields of every resource are always `url` and `id`. They both uniquely identify the resource. You may add the `id_only=true` condition to the query part of the url if you don't wish to receive the `url` field(s). This option will also apply to resource references. The `url` fields are shown by default to increase explorability of the API. If you are creating an automated import then you probably want to add `id_only=true` to all your requests.

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
* Authentication / versioning / TLS
* Explain time format and link to iso (UTC only).
* Explain default CORS setting.
* Explain item sets
* Explain comma seperated export (by adding .txt or .csv to resource).
