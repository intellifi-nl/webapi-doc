Resources
=========

This document gives a detailed description of the avaialble resources. You can find an overview description in the [Brain web API documentation](README.md).

Contents
--------

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

Items
-----

The items resource will contain all [item](README.md#item) objects that where detected by one of the [zones](README.md#zone). They are automatically added as soon as they are are detected for the first time. The items resource couples a unique id to every item and gives a place to add more information to an item.

Every item contains at least a unique id (`item_id`), the detected `code` and the `codetype_mask`. You may add a `label` and an `image` to the item. The `item_id` is the reference to the item that is used in all other places in the system.

An item will also contain the current (`location_now`) and last known location (`location_now`) of an item. This is the output of the localisation service. The `location_now` will be set to null when item is not actively detected, `location_last` will keep the last known location for as long as the item exists. In most use cases these 2 small fields will just tell you all the things that you want to know: Where is my item (`location_now`)? Or where has it been seen for the last time (`location_last`)?

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `code` | string | String representation of the unique code that this item transmits. By default this is a hexadecimal representation. This number can be so long (> 40 bytes!) that a decimal representation would be useless to generate.
| `code_type`  | string | Type of technology that was used to detect this item. Can be 'EPC Gen2', 'Bluetooth LE' or 'iBeacon'. |
| `label`| string | A name or a label for this item. You may add a number or id for your own system. |
| `location_now` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to a [location](#locations) if the item as actively detected on some place. Null when the item is not beeing detected. |
| `location_last` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Almost the same as `location_now`. Is **not** configured to null when object is not detected anymore. |
| `last_location_event` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the last event that updated the location. We should also add url field to this so that you can follow it. |
| `time_create` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource created? |
| `time_update` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource updated for the last time? Can be a location update or a label update. At some point we will add an option that can delete items that have not been updated for a to long time... |

The `code_type` always contains a single string to keep things simple. It's important to state that an [iBeacon](http://en.wikipedia.org/wiki/IBeacon) is a Bluetooth LE transponder as well. We always show the most specific type in this field.

You may be worried about the amount of items that could flow into your system. In the future you can configure the spots to only allow certain code ranges with the flexible item sets approach. With this approach you can filter the amount of tags that come into your system. It will also become possible to 'drop' items after a certain amount of time (off course this shall only apply to items that you didn't edit).

Ideas:
* `image` field: so that you can save an optional picture with an item. Could be handy for simple front end app without own storage.
* Add extra timestamp with the last time that the location was updated. Could be handy to see changes in your warehouse.

Spots
-----

The Intellifi Spot devices form the eyes and the ears of the server logic. They generate events for every item that is detected. By doing so they 'implement' the earlier described [zone](README.md#zone) abstraction. 

Every spot has it's own representation inside the spots resource. This allows you to see and monitor the current status of a spot. You can also configure the location that the spot reports it's detections to.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to location resource that this overall spot reports it's detection to. You may set this to null if you don't want the spot to report overall presences. |
| `serial_number` | number | This is the fixed and unqiue spot id, as used in the embedded device. |
| `is_online` | boolean | True when the spot is active and capable of sending events. |
| `state` | string | The current state of the spot. |
| `request_counter` | number | The total number of HTTP requests that the spot has done |
| `time_last_request` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The timestamp of the last received HTTP request to this server |
| `received_spot_object` | object | An object with specific information about the spot, directly send by the spot itself when the connection is created. |
| `received_spot_config` | object | An object with the current spot configuraton, also directly sned by the spot itself when the connection is created. |
| `time_create` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. The timestamp of the first HTTP request to this server. |
| `time_update` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? Is not updated by the request_counter, you can use time_last_request to see that. |

You can't add a label or a note to the spot. This is by design. We gave the seperate [location resource](#locations) responsibility for labels and notes.

The spots are automatically added to this resource when they are connected to this server. Spots are never deleted automatically, you may delete a spot that is offline with the HTTP delete action.

We will include a way to authenticate a spot in the future. This is a critical feature to refrain people from abusing the openness of our system. At this moment we advise you to use production spots in a 'closed' network only. Please let us know if you are very eager to have this feature.

Antennas
--------

All Intellifi Spots contains one or more antennas, you can even connect external antennas. This resource can be used to query all available antennas in your system. You may let a single antenna report to location. Doing so effectively defines an extra zone, or virtual spot.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `spot` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | To which Spot is the antenna connected? |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | By default null, may be set to a valid `location_id` |
| `antenna_number` | number | Internal number as used in the spot. Starts counting at 1. For smart antennas we are probably going to have a longer unique number. Or shall we have a seperate serial_number field? |
| `time_create` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |
| `time_update` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? |

You also don't add a label or location here as well. You can define it in the location that the antenna might report to.

The antennas are created automatically when the spot is connected.

You can edit this resource by posting your change to it (elaborate on this).

Locations
---------

The locations resource allows you to create, read and update the definitions for your locations. A location couples a [zone](README.md#zone)(i.e an Intellifi Spot) to a label and an optional geographic position.

A zone can 'trigger' a location. If a zone detects an item then it allows the connected location to perceive. In most situations your Intellifi Spot will behave itself as a single zone and will trigger a single location. An antenna that is connected to the Intellifi spot may also be used as a zone (virtual spot). And thus can also trigger a location. The location itself can also behave like a zone. It can 'forward' the received events to it's optionally configured parent (`report_location` field). This is a powerfull concept that allows you to layer your location definitions. The kitchen, hall and livingroom could report to a house locaton for example.

Locations are coupled by a bottum-up approach. Every zone may report to a single locaton. Every location may report to one another location (repeat this sentice as many time as you wish). You may let different zones report to a single location. Effectively this will allow you to filter your information. With the [presence resource](#presences) you can query presences on a certain location. In example: if you are only intrested in the people that are currently in your house then you can just query for that.

These concepts allow you to use the Intellifi Spots in a lot of situations. You can let multiple spots report to a single big location. Effectively a bigger zone. You can also let one Spot report to several smaller locations (virtual spot). Every antenna can be used as a seperate zone.

A default location for an Intellifi Spot is automatically created when you connect it to the server for the fist time. You may edit or remove this location. All connected antennas are automatically created as well, they are not configured to a location by default. The overal spot will report for them.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to location that this locaton reports to, null by default. In a way it's a parent location. |
| `label` | string | How do you name this resource? Or how do you refer to it in your own applicaton?
| `hold_time_s` | number | How long should an item be kept present at this location? |
| `time_create` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |
| `time_update` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? |

Ideas:
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
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `item` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the item that was detected |
| `location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the locaton that this item was seen on. |
| `proximity` | string | Strongest proximity of all 'child' presences, see next paragraph. |
| `time_create` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | Created time, when was the first hit received for this presence? |
| `time_update` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last edit in this presence resource? |

We add a estimated proximity to every presence. This is a rough estimate on the distance from the item to the receiver. 3 possible values are returned:

1. 'far': the item is detected, but the received signal is weak. In most cases this means that the item is far away, but it also might indicate that you have interference or a seriously low battery.
2. 'near': the item is detected with an average signal strength.
3. 'immediate': the item is detected with a very strong signal. It must be very close to your antenna.

The returned value depends on the configured signal levels. It's possible to adjust these levels to your situation, please refer to the detailed Intellifi spot documentation if you would like to do this.

Ideas for extra fields:
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
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `resource_type` | string | One of the defined [resources](#resource). Is also written in it's plural form. I.e. 'spots', 'items'. |
| `resource` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to one of the existing resources. |
| `resource_id_short` | number | May be filled with a shorter identifying number (null when not filled). I.e. The spots resource always puts the spot `serial_number` into this field.
| `action` | string | Indicates the kind of event that was executed. In most cases it's a verb. I.e. 'connect', 'create' etc. Multiple levels may be avaialble, they are seperated by a dot (AMQP style).|
| `time_create` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this event resource created at the server? |
| `time_device` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When did this event actually took place on the device? This is the device it's own timestamp. Could be different due to buffering and clock differences. |
| `payload` | object | An object that contains the used encoding and the actual payload (if any). We will try to get this in line with the websockets output. Possible encodings: 'json', 'utf8' or 'base64'. We might add 'null', for now it's just an empty utf8 string if nothing was send.  |

All the event resources have been published on the message bus at some time. You might be wondering where the `topic` field went. It's actually parsed into the `resource-type`, `resource`, `resource_id_short` (when applicable) and `action` fields. You can find more information in the  [topic format](https://github.com/intellifi-nl/doc-push/blob/master/mqtt_topics.md#format) that is described in the push documentation. 

Be carefull with the given `time_create` and `time_device` fields. Intellifi Spots can buffer events in the case of a network loss (a very small number for the moment). If you are traversing over the resource then you should look at `time_create`, eventual old events that pop up will just be in your result. If you are actually doing something with the event then you should look at `time_device`!

An event is never changed (can't be by definition!), so we don't offer a `time_update` field on this resource.

Ideas:
* Add url to location change event of previous change. So that we can walk through the location updates. You could also request them by the right query. Perhaps we should also include that at a place?
* It might be very handy to allow the end user to 'subscribe' this table with a set of topics. It limits system load and allows us to keep sensible events arround longer. Or should we move this to a seperate resource (filtered_events, subscribed_events)? For now we will just subscribe to all events.
