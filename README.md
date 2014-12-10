Web API documentation
=====================

This document contain the offical web API documentation. We provide a RESTful API that allows you to access all the data that we provide in a powerfull and simple way.

At this moment we only support JSON as output format.

By default the api is accessible on: http://`host`/api/`resource`/`id`
* The `host` will be provided to you when you are evaluating or purchasing our product. We always have an 'play-arround' brain that we can supply to you.
* The `resource` shall will contain the resource that you want to query. This is most of the times the plural form of a noun.
* The optional `id` indicates which specific resource you wish to access. Please refer to the individual resourcse for more information on the type of id that is used. In most resources this is a [MongoDB ObjectId](http://docs.mongodb.org/manual/reference/object-id/). If you omit `id` the server will return a list with all items in the resource.

As with every web API you can only request new information by doing an extra request. We offer a whole set of [pushing technologies](https://github.com/intellifi-nl/doc-push). They will allow you to be informed when something changes, instead of polling for changes.

Contents
--------

* [Terminology](#terminology)
  * [Item?](#item)
  * [Zone?](#zone)
* [Explorability](#explorability)
* [Resources](#resources)
  * [Items](#items)
  * [Spots](#spots)
  * [Locations](#locations)
  * [Presences](#presences)
  * [Paths](#paths)
  * [Passings](#passings)
  * [Sets](#sets)
  * [Senses](#senses)
  * [Events](#events)
* [Pagination](#pagination)
* [Todo](#todo)

Terminology
===========

The whole Intellifi concept is based upon items and zones. Please take some time to familiarize with these definitions. They will make it way easier to understand this API.

Item
----

An item is a verry little electronic device that contains and remembers a unique code. An item must be able to transmit this unqiue code wireless. This item may be a passive RFID tag, but it can also be an iBeacon. Even your smartphone could behave itself as an item. A device is an item as long as it's able to remember and transmit it's unique code.

Zone
----

A zone is an area in which items can be detected. This area is naturually limited to the range of the used RFID technology. Passive tags have a range of approximately 10 meters, active tags can easily have a range of 100+ meters. A zone must be able to reports it's detected items.

A lot of devices are capable of behaving themselves as 'zones'. Our [Intellif Spot](http://intellifi.nl/home/products/) is a very good example of this. It can detect RFID tags and Bluetooth tags that are in the neighbourhoud. It reports these detections through it's network interface to a server. It's also possible to connect external antennas to the Spot. By default they are used to enlarge the reach of the overall Spot zone. You may configure individual external antenna to behave themselve as a zone as well. By doing so you are essentially creating a virtual spot. 

Another great example of a zone could be your smartphone. Lot's of smartphones support the detection if iBeacons. It's a matter of the right app to report this information to a server. And voilà: here's another zone that you can use to detect your items.

A zone is an abstraction and is not avaialble as a resource. Please make sure to read more about the locatons resource, there we will 'connect' the zone to the rest of the description.

Explorability
=============

We find it very important that our web API is self explaining. We strongly recommand you to install a JSON viewer plugin in your webbrowser. This will allow you to view the query results in your web browser. For Google Chrome we advice you to use [JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc). Without doubt there will be a nice plugin for your own personal browser as well.

Most of the resources include links to other relevant resources. These links are added as fields to the JSON objects. A good JSON viewer will allow you to follow them with a simple click. These fields always have the "url_" prefix.

Resources
=========

Items
-----

The items resource will contain all RFID tags and Bluetooth LE (BLE) transponders that are detected by the Intellifi spots. They are automatically added as soon as they are are detected for the first time. The items resource is an abstraction that allows you to work with RFID tags and BLE transponders as if they where the same.

Every item contains at least a unique id, the `code` and the `codetype_mask`. You may add a label to the item. The item_id is the reference to the item that is used in all other places in the system.

* `item_id`
* `code`
* `codetype_mask`
* `label`
* `image`
* `location_now`
* `location_last`
* `time_first`
* `time_last`

The `codetype_mask` is a bitwise number that allows you to identify the kind of technology that was detected. Within bluetooth it's possible to detect multiple types at the same time. i.e. A Bluetooth LE transponder may be an iBeacon. We might add support to other Bluetooth profiles in the future, we also could support other technologys at some point. For now we have this list with types:

* 1 RFID tag (EPC Gen2)
* 2 Bluetooth LE transponder: indicates that you shall have attributes to read full PDU and MAC address.
* 4 [iBeacon](http://en.wikipedia.org/wiki/IBeacon): indicates that you shall have UUID, major and minor. Not yet supported, may change.
* 8 [BLE Proximity profile](http://stackoverflow.com/questions/16952185/what-exactly-is-the-proximity-profile-with-respect-to-the-bluetooth-low-energy): indicates that you can read the RSSI value that is received on the transponder, after connecting to it. Not yet supported, may change.
* 16 [BLE Find me profile](https://developer.bluetooth.org/TechnologyOverview/Pages/FMP.aspx): indicates that you can request an alert sounding on a specific item. Not yet supported, may change.
* 32 [BLE Battery profile](https://developer.bluetooth.org/TechnologyOverview/Pages/BAS.aspx): Will allow you to know the current battery status of the item.

Other [profiles](https://developer.bluetooth.org/TechnologyOverview/Pages/Profiles.aspx) may be supported in the future as well.

You may be worried about the amount of items that could flow into your system. In the future you can configure the spots to only allow certain code ranges with the flexible item sets approach. With this approach you can filter the amount of tags that come into your system. It will also become possible to 'drop' items after a certain amount of time (off course this shall only apply to items that you didn't edit).

Spots
-----

The Intellifi sports form the eyes and the ears of the server logic. They generate events for every item that is detected. Every spot has it's own representation inside the spots resource. This allows you to see and monitor the current status of a spot.

* spot_id: This is the fixed and unqiue spot id, it's the only id in this API that is not an MongoDB ObjectId.
* is_online: True when the spot is active and capable of sending events.
* state: The current state of the spot.
* request_counter: The total number of HTTP requests that the spot has done.
* time_first_request: The timestamp of the first HTTP request to this server.
* time_last_request: The timestamp of the last received HTTP request to this server.
* received_spot_object: An object with specific information about the spot, directly send by the spot itself when the connection is created.
* received_spot_config: An object with the current spot configuraton, also directly sned by the spot itself when the connection is created.
* report_location: May be null, may contain an location id to 

At this moment you can't add a label or a note to the spot. We created the seperate location resource for this purpose.

The spots are automatically added to this resource when they are connected to this server. Spots are never deleted automatically, you may delete a spot that is offline with the HTTP delete action.

Locations
---------

The locations resource allows you to create, read and update the definitions for your locations. A location couples Intellifi spots to a geographic position and label.

In a way the Intellifi spot 'triggers' an location. If a spot detects an item then it allows the location to perceive. An antenna that is connected to the Intellifi spot may also trigger a location (this is also called a virtual spot). It's even possible to define other locations as trigger. You must define 1 or more triggers on a location. You can use as many triggers as you like. This powerfull concept allows you to define multiple locatons with one spot, or on the other hand: multiple spots in one bigger location.

A default location for a Intellifi Spot is automatically created when you connect it to the server for the fist time. You may edit or remove this location. You may also use the Intellifi spot in multiple location definitions. They all will be triggered when the Intellifi spot detects items.

* location_id
* triggers
* label
* hold_time_s (how long should an item be present at this location?)
* x, y
* picture
* building_map (only when you are defining an overlapping location)

Presences
---------

An item can be present on a location. A presence resource is automatically created when one of the defined location triggers says that an item is detected. A presence is deleted when it has not been detected for n seconds. Where n is the hold time in seconds. So the presence resource exactly tells you where your items are beeing detected at this very moment!

An item can be present on multiple locatons at the same time. This is caused by two main reasons:

1. You may define locatons that are triggered by another location. I.e. your office building could be triggered by the hall, kitchen that it contains. It's logical that you can be present in the kitchen and in your office building at the same time.
2. The used technolgy have a great range. Your items may be picked up by multiple devices at the same moment. We will present all this information to you.

We add a estimated proximity to every presence. This is a rought estimate on the distance from the item to the receiver. 3 possible values are returned:

1. far: the item is detected, but the received signal is weak. In most cases this means that the item is far away, but it also might indicate that you have interference or a seriously low battery.
2. near: the item is detected with an average signal strength.
3. immediate: the item is detected with a very strong signal. It must be very close to your antenna.
The returned value depends on the configured signal levels. 

> It's possible to adjust these levels to your situation, please refer to the detailed Intellifi spot documentation if you would like to do this.

If you added multiple triggers to a location then the strongest proximity is returned in the created presence.

If you just only want to know where something is located then we have good news: we already did the hard work for you. The location service does a best fit and determines where your item is. The calculated location is directly saved within the items resource.

Events
------

The events resource keeps a copy of events that occured. This is an exact copy of the events that are avaialble on the message bus. Please note that lots of events are flowing through the system. The history of events is kept for a limited time. If you would like to retreive all events then you should consider conneting to our message bus through one of the avaialbe [push technologies](https://github.com/intellifi-nl/doc-push).

Every event is envelopped in an JSON object with the following fields:
* resource: One of the strings that we defined in the resources chapter. i.e. spots
* id: A valid id for the resource that you choose. i.e. 203
* action: A string that indicates the action that was executed. In most cases it's a verb. i.e. connect
* object: A JSON object with extra information about the event, or the actual resource if something changed.
* time: An event always takes place at a fixed time.

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
* Versioning (/v2/)
* CORS
* Explain time format and link to iso.
* TODO: State vs events (seperate access to message bus and websocket)
* TODO: https is not yet supported, use http!
