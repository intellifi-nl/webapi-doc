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
  * [Items](#items)
  * [Spots](#spots)
  * [Antennas](#antennas)
  * [Locations](#locations)
  * [Presences](#presences)
  * [Paths](#paths) (not yet)
  * [Passings](#passings) (not yet)
  * [Sets](#sets) (not yet)
  * [Senses](#senses) (not yet)
  * [Events](#events)  
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

Items
-----

The items resource will contain all [item](#item) objects that where detected by one of the [zones](#zone). They are automatically added as soon as they are are detected for the first time. The items resource couples a unique id to every item and gives a place to add more information to an item.

Every item contains at least a unique id (`item_id`), the detected `code` and the `codetype_mask`. You may add a `label` and an `image` to the item. The `item_id` is the reference to the item that is used in all other places in the system.

An item will also contain the current (`location_now`) and last known location (`location_now`) of an item. This is the output of the localisation service. The `location_now` will be set to null when item is not actively detected, `location_last` will keep the last known location for as long as the item exists. In most use cases these 2 small fields will just tell you all the things that you want to know: Where is my item? Or where has it been seen for the last time?

| Field | Type | Description |
| ----- | ---- | ----------- |
| `item_id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `code` | string | String representation of the unique code that this item transmits. By default this is a hexadecimal representation. These number can be so long (> 40 bytes!) that a decimal representation would be useless to generate.
| `codetype_mask`  | number | Bitwise number that allows you to identify the kind of technology that was detected. |
| `label`| string | A name or a label for this item. You may add a number or id for your own system. |
| `location_now` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to a [location](#locations) if the item as actively detected on some place. Null when the item is not beeing detected. |
| `location_last` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Almost the same as `location_now`. Is **not** configured to null when object is not detected anymore. |
| `last_location_event` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the last event that updated the location. We should also add url field to this so that you can follow it. |
| `time` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource created? |

The `codetype_mask` field allows you identify the kind of technology that was used to detect the tag.
It's a bitwise number because multiple technologies can be used at the same time: i.e. A Bluetooth LE transponder may be an iBeacon. A bitwise field required you to sum the individual values, so in case of an iBeacon you sum Bluetooth LE transponder (2) and iBeacon (4). The value would be 2 + 4 = 6.

* 1 RFID tag (EPC Gen2)
* 2 Bluetooth LE transponder: indicates that you shall have attributes to read full PDU and MAC address.
* 4 [iBeacon](http://en.wikipedia.org/wiki/IBeacon): indicates that you shall have UUID, major and minor.

You may be worried about the amount of items that could flow into your system. In the future you can configure the spots to only allow certain code ranges with the flexible item sets approach. With this approach you can filter the amount of tags that come into your system. It will also become possible to 'drop' items after a certain amount of time (off course this shall only apply to items that you didn't edit).

Idears:
* `image` field: so that you can save an optional picture with an item. Could be handy for simple front end app without own storage.
* Add extra timestamp with the last time that the location was updated. Could be handy to see changes in your warehouse.

Spots
-----

The Intellifi Spots form the eyes and the ears of the server logic. They generate events for every item that is detected. By doing so they 'implement' the earlier described [zone](#zone) abstraction. 

Every spot has it's own representation inside the spots resource. This allows you to see and monitor the current status of a spot. You can also configure the location that the spot reports it's detections to.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `spot_id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `serial_number` | number | This is the fixed and unqiue spot id, as used in the embedded device. |
| `is_online` | boolean | True when the spot is active and capable of sending events. |
| `state` | string | The current state of the spot. |
| `request_counter` | number | The total number of HTTP requests that the spot has done |
| `time` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. The timestamp of the first HTTP request to this server. |
| `time_last_request` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The timestamp of the last received HTTP request to this server |
| `received_spot_object` | object | An object with specific information about the spot, directly send by the spot itself when the connection is created. |
| `received_spot_config` | object | An object with the current spot configuraton, also directly sned by the spot itself when the connection is created. |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to location resource that this overall spot reports it's detection to. You may set this to null if you don't want the spot to report overall presences. |

You can't add a label or a note to the spot. This is by design. We gave the seperate [location resource](#locations) responsibility for labels and notes.

The spots are automatically added to this resource when they are connected to this server. Spots are never deleted automatically, you may delete a spot that is offline with the HTTP delete action.

We will include a way to authenticate a spot in the future. This is a critical feature to refrain people from abusing the openness of our system. At this moment we advise you to use production spots in a 'closed' network only. Please let us know if you are very eager to have this feature.

Antennas
--------

All Intellifi Spots contains one or more antennas, you may also connect external antennas to a Spot. You can use this resource to query all available antennas in your system. You may let a single antenna report to location. Doing so effectively defines an extra zone, or virtual spot.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `antenna_id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `spot` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | To which Spot is the antenna connected? |
| `antenna_number` | number | Internal number as used in the spot. Starts counting at 1. For smart antennas we are probably going to have a longer unique number. Or shall we have a seperate serial_number field? |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | By default null, may be set to a valid `location_id` |
| `time` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |

You also don't add a label or location here as well. You can define it in the location that the antenna might report to.

The antennas are created automatically when the spot is connected.

You can edit this resource by posting your change to it (elaborate on this).

Locations
---------

The locations resource allows you to create, read and update the definitions for your locations. A location couples a [zone](#zone)(i.e an Intellifi Spot) to a label and an optional geographic position.

A zone can 'trigger' a location. If a zone detects an item then it allows the connected location to perceive. In most situations your Intellifi Spot will behave itself as a single zone and will trigger a single location. An antenna that is connected to the Intellifi spot may also be used as a zone (virtual spot). And thus can also trigger a location. The location itself can also behave like a zone. It can 'forward' the received events to it's optionally configured parent (`report_location` field). This is a powerfull concept that allows you to layer your location definitions. The kitchen, hall and livingroom could report to a house locaton for example.

Locations are coupled by a bottum-up approach. Every zone may report to a single locaton. Every location may report to one another location (repeat this sentice as many time as you wish). You may let different zones report to a single location. Effectively this will allow you to filter your information. With the [presence resource](#presences) you can query presences on a certain location. In example: if you are only intrested in the people that are currently in your house then you can just query for that.

These concepts allow you to use the Intellifi Spots in a lot of situations. You can let multiple spots report to a single big location. Effectively a bigger zone. You can also let one Spot report to several smaller locations (virtual spot). Every antenna can be used as a seperate zone.

A default location for an Intellifi Spot is automatically created when you connect it to the server for the fist time. You may edit or remove this location. All connected antennas are automatically created as well, they are not configured to a location by default. The overal spot will report for them.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `location_id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `label` | string | How do you name this resource? Or how do you refer to it in your own applicaton?
| `hold_time_s` | number | How long should an item be kept present at this location? |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to location that this locaton reports to, null by default. In a way it's a parent location. |
| `time` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |

Idears:
* x, y coordinates so that you can draw a map of locations.
* picture of a location.

Presences
---------

An item can be present on a location. A presence resource is automatically created when one of the defined location triggers says that an item is detected. A presence is deleted when it has not been detected for n seconds. Where n is the hold time in seconds. So the presence resource exactly tells you where your items are beeing detected at this very moment.

An item can be present on multiple locatons at the same time. This is caused by two main reasons:

1. The used technolgies all have a big range. Your items may be picked up by multiple devices at the same moment. In this resource we present all this information to you.
2. You may define locatons that are triggered by another location. I.e. Your 'office room 3' and 'hall' locaton could both report to your overall 'office building' location. It's logical that you can be present in 'office room 3' and in your 'office building' at the same time.

If you just only want to know where something is excatly located then we have good news: we already did the hard work for you. The localisation service does a best fit and determines where your item is. The calculated location is directly saved within the [items](#items) resource. You don't need to query this resource in that situation. This resource reveals more of wat is happening inside the system. For some use cases this is really usefull.

Presence are deleted when their hold time expires, or in other words: when they have not been seen for a some time. This is an important difference to the first API version that we had. You can use the [events resource](#events) to query all events that took place in the system. Including create, update and delete events for presences. So you can always reconstruct the presences that where avaialble at some time. Please let us know if it would make you happy that we did this for you.

The presence contains these fields:

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `presence_id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `item` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the item that was detected |
| `location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the locaton that this item was seen on. |
| `proximity` | string | Strongest proximity of all 'child' presences, see next paragraph. |
| `time` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | Created time, when was the first hit received for this presence? |
| `time_last` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last edit in this presence resource? |

We add a estimated proximity to every presence. This is a rough estimate on the distance from the item to the receiver. 3 possible values are returned:

1. 'far': the item is detected, but the received signal is weak. In most cases this means that the item is far away, but it also might indicate that you have interference or a seriously low battery.
2. 'near': the item is detected with an average signal strength.
3. 'immediate': the item is detected with a very strong signal. It must be very close to your antenna.

The returned value depends on the configured signal levels. It's possible to adjust these levels to your situation, please refer to the detailed Intellifi spot documentation if you would like to do this.

Idears for extra fields:
* parent: parent id of presence.
* children: ids of all children that 'feed' this presence. Should this also include spots or antennas?
* url_parent: Direct link so that you can explore.
* url_children: Direct links so that you can explore.
* url_hits: link to events resource with all hits for this presence.

Events
------

The events resource keeps a copy of all events that occured. This is an exact copy of the events that are avaialble on the message bus. Please note that lots of events are flowing through the system. The history of events is kept for a limited time. If you would like to retreive all events then you should consider connecting to our message bus through one of the avaialble [push technologies](https://github.com/intellifi-nl/doc-push).

Every event is envelopped in an JSON object with the following fields:

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `event_id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `resource` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to id of one of the existing resources. |
| `resource_type` | string | One of the defined [resources](#resource). Is also written in it's plural form. I.e. 'spots', 'items'. |
| `action` | string | Indicates the kind of event that was executed. In most cases it's a verb. I.e. 'connect', 'create' etc. |
| `time` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this event resource created at the server? |
| `time_event` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When did this event actually took place on the device? This is the device it's own timestamp. Could be different due to buffering and clock differences. |
| `payload` | object | A JSON object with extra information about the event, or the actual resource if something changed. |

All the event resources have been published on the message bus at some time. You might be wondering where the `topic` field went. It's actually parsed into the `resource-type`, `resource` id and `action` fields. You can find more information in the  [topic format](https://github.com/intellifi-nl/doc-push/blob/master/mqtt_topics.md#format) that is described in the push documentation. 

Be carefull with the given `time` and `time_event` fields. Intellifi Spots can buffer events in the case of a network loss (a very small number for the moment). If you are traversing over the resource then you should look at `time`, eventual old events that pop up will just be in your result. If you are actually doing something with the event then you should look at `time_event`!

Idears:
* Add url to location change event of previous change. So that we can walk through the location updates. You could also request them by the right query. Perhaps we should also include that at a place?

Design
======

Import API design decisions are described in this chapter. You will find out why we do some things.

Explorability
-------------

We find it very important that our web API is self explaining. We strongly recommand you to install a JSON viewer plugin in your webbrowser. This will allow you to view the query results in your web browser. For Google Chrome we advice you to use [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own personal browser as well.

Most of the resources include links to other relevant resources. These links are added as fields to the JSON objects. A good JSON viewer will allow you to follow them with a simple click. These fields always have the "url_" prefix.

Authentication
--------------

API keys, users etc.

Pagination
----------

The number of results is always limited to 100. Obviously we do allow you to make more querys so that you can retreive the rest of the results. This process is called paginiation and keeps our server load at acceptable levels.
* Default list/listing envelope
* Links that help you
* TODO: Implement RFC specific headers?
* TODO: Be carefull when resources are added or deleted during a paginiation. If you don't want to miss a thing then the events resource is the only reliable source.

Versioning
----------

/always the latest versions?
/v2/?

Todos
-----

* CORS
* Explain time format and link to iso.
* https is not yet supported.
* expand for releations.
* option for extra navigational url's?
* Time, UTC only on our side. Presentation layer can add a timezone.
* Move the resources to a seperate page and give only an overview on this page.
