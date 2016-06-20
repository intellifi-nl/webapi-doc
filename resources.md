Resources
=========

This document gives a detailed description of the available resources. You can find an overview description in the [Brain web API documentation](README.md).

Contents
--------

* [Items](#items)
* [Sets](#sets)
* [Locations](#locations)
* [Spots](#spots)
* [Presences](#presences)
* [Subscriptions](#subscriptions)
* [Events](#events)
* [Services](#services)

Items
-----

The items resource will contain all [item](README.md#item) objects that where detected in one of the [zones](README.md#zone). They are automatically added as soon as they are detected for the first time. The items resource couples a unique id to every item and gives a place to add more information to an item.

Every item contains at least a unique id (`id`), the detected `code` and the `technology` (this combination is unique). The `item_id` is the reference to the item that is used in all other places in the system.

You may add a `label` and a `custom` value to the item. The label is used to show a human readable name on several places in our user interface. 

The item is also a placeholder for the output of the localization service. If the item is being detected on multiple places then we offer a most likely `location`, based on all available information. The `location` only changes when the item moves to another place. If it moves out of reach then it sticks to the last known `location`. You can use `is_present` to see if the item is actively detected. We believe that this information is enough for most use cases, it just tells you where your items are.

Every item has the following fields defined:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `code_hex` | string | String representation of the unique code that this item transmits. By default this is a hexadecimal representation. This number could be so long (> 40 bytes!) that a decimal representation would be useless to generate.
| `technology`  | string | Type of technology that was used to detect this item. Can be 'EPC Gen2' or 'Bluetooth LE'. |
| `label`| string | A name or a label for this item. Is shown in our user interface, may also be empty. |
| `custom` | value | The `custom` value is only for your own reference, you may use it to save additional attributes. The `custom` value is not used on any other place. This field may contain any datatype that you like: null (default), text, number, boolean or object. |
| `set_urls` or `set_ids` | array | An array with referenecs to the sets that this item is in. set_urls is not implemented at the time of writing, you should set id_only=true to retreive an array with `id` strings.  |
| `is_present`| boolean | Is this item actively detected in one of the zones at this moment? True when it is, false if it's not. |
| `location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to the [location](#locations) resource where the item is located. Or, if the item is out of reach, the last known location. |
| `move_count` | value | How many times was this item moved since it's created on this server? Please note that it is likely that not all moved events are available in the events (they are deleted after a configurable number of hours). This field is never decreased. It gives a nice indication of the usage of this item. |
| `time_moved` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this item moved for the last time? This is the last time that the location was changed. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource created? |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource updated for the last time? Can be a location update or a label update. |

At this moment it's not possible to delete items.

A special sub-resource is available to request all available item moves (it's a view on the events resource): /api/items/moves
* At this moment you can only populate the labels, ?populate=item.label,location.label
  * The `url` and `id` are always included as well (unless you add id_only=true).
  * This specific dot notation is not supported in other resources at this moment.
  * It could be usefull to enable the `auto_label_with_code_hex` in the Engine service. This allows you to retreive the code_hex values without doing lookups.
  * You may filter on a specific location or item id. /api/items/moves?item=5698f927e7831505f13a4aea

Idears
* You may be worried about the amount of items that could flow into your system. We are thinking about a way to configure SmartSpots to only allow certain code ranges with the flexible item sets approach. With this approach you can filter the amount of tags that come into your system.
* We are also thinking about a smart way to delete unused items after a certain amount of time.
* We are thinking about parsing the iBeacon and 
* Please let us know if you like these idears, or if you have additional thoughts.

Sets
----

Items can be grouped together into a set. At this moment we only support lists, later we might be adding support for i.e. masks.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `type` | string | What kind of set is this? At this moment we only support 'list'. |
| `resource_type`  | string | What resource is this set combining? At this moment we only support 'items'. |
| `label`| string | A name or a label for this set. Is shown in our user interface, may also be empty. |
| `custom` | value | The `custom` value is only for your own reference, you may use it to save additional attributes. The `custom` value is not used on any other place. This field may contain any datatype that you like: null (default), text, number, boolean or object. |
| `terms`| object | An object that contains the specific properties for this set, it depends on the type and resource_type which fields will be available. |
| `view_url`| string | A collection with items that are currently in this set. |
| `is_syncable_to_spot`| boolean | Is not supported at this moment, is always false. Later we could sync sets to the SmartSpot hardware. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? |

As said, at this moment we only have support for a item list, the terms object of these sets contains the following parameters:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `ids_url` | string | Url that you can use to `POST` and `DELETE` item id's to and from. At this moment there is no support for `GET`, use the view_url instead. |
| `allow_add`  | boolean | Not yet supported: is it allowed to perform this action? |
| `allow_remove`  | boolean | Not yet supported: is it allowed to perform this action? |
| `allow_remove_all` | boolean | Not yet supported: is it allowed to perform this action? |
| `allow_local_add` | boolean | Not yet supported: is it allowed to perform this action? |
| `allow_local_remove` | boolean | Not yet supported: is it allowed to perform this action? |
| `allow_local_remove_all` | boolean | Not yet supported: is it allowed to perform this action? |
| `capacity_limit` | value | Not yet supported: How many items may you add to this set? |

Locations
---------

The locations resource allows you to create, read and update the definitions for your locations. A location couples a [zone](README.md#zone)(i.e an Intellifi Spot) to a `label` and `custom` field that you can fill in with any information that you like.

A zone or reader device reports to a location. In most situations your Intellifi Spot will behave itself as a single zone and will report to a single location. An antenna that is connected to the Intellifi spot may also be used as a zone (virtual spot). And thus can also trigger a location. Locations are coupled by a bottum-up approach. Every zone may report to a single locaton. You may let different zones report to a single location.

This concept allows you to use the Intellifi Spots in a lot of situations. You can let multiple spots report to a single big location. Effectively a bigger zone. You can also let one Spot report to several smaller locations (virtual spot). Every antenna can be used as a seperate zone.

A default location for an Intellifi Spot is automatically created when you connect it to the server for the fist time. You may edit or remove this location.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `label` | string | How do you name this resource? Or how do you refer to it in your own applicaton?
| `custom` | value | The `custom` value is only for your own reference, you may use it to save additional attributes. The `custom` value is not used on any other place. This field may contain any datatype that you like: null (default), text, number, boolean or object. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? |

Spots
-----

The Intellifi Spot devices form the eyes and the ears of the server logic. They generate events for every item that is detected. By doing so they 'implement' the earlier described [zone](README.md#zone) abstraction. 

Every spot has its own representation inside the spots resource. This allows you to see and monitor the current status of a spot. You can also configure the location that the spot reports its detections to.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `report_location` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Reference to location resource that this overall spot reports its detection to. You may set this to null if you don't want the spot to report overall presences. |
| `antenna_report_locations` | object | You may configure this field to an object which couples individual antenna ports to locations. An example is given below. |
| `serial_number` | number | This is the fixed and unqiue spot number. It's assigned during the production process and used to identify an individual device during its lifetime. |
| `is_online` | boolean | True when the spot is active and capable of sending events. |
| `state` | string | A text string indicating the current state of the spot. At this moment 'active' is the only value that yuo should see. |
| `request_counter` | number | The total number of HTTP requests that the spot has done |
| `status` | object | An object with specific information about the spot, directly send by the spot itself when the connection is created. |
| `config` | object | An object with the current spot configuraton, also directly send by the spot itself when the connection is created. |
| `config_request` | object | You can request a config change by doing a PUT on this field. You should put the same name is in the `config` object with the value that you would like. The request is forwarded to the spot, and then the setting is applied (when valid). The field is cleared as soon as the request is transferred to the spot. |
| `senses` | object | Senses are values that in most cases are generated inside the spot (number of presences, spot booted etc.). We also have a few senses that can be controlled by the brain. See [update remote sense](#update-remote-sense) for more info. |
| `senses_request` | object | You can request a change for a brain sense by doing a PUT on this field. As soon as the request is handled the field is cleared. See [update remote sense](#update-remote-sense) for more info.
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | The time that this resource was created. The timestamp of the first HTTP request to this server. |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change in this resource? Is not updated by the request_counter, you can use time_last_request to see that. |

You can't add a label or a note to the spot. This is by design. We gave the seperate [location resource](#locations) responsibility for labels and notes.

The spots are automatically added to this resource when they are connected to this server. Spots are never deleted automatically, you may delete a spot that is offline with the HTTP delete action.

We will include a way to authenticate a spot in the future. This is a critical feature to refrain people from abusing the openness of our system. At this moment we advise you to use production spots in a 'closed' network only. Please let us know if you are very eager to have this feature.

### Defining report locations

Every spot can report its presences to one location. You can also configure individual antennas to report to different locations (antenna presences). You may do an HTTP PUT with the following body to configure 4 individual antennas to report to some location. You have to supply the id's of the locations that you want to report to.
```
{
	"report_location": "54f97c3cc573f4a82099749f",
	"antenna_report_locations":[
        {"antenna_number":1, "report_location": "54f97c3cc573f4a82099749c"},
        {"antenna_number":2, "report_location": "54f97c3cc573f4a82099749c"},
        {"antenna_number":3, "report_location": "54f97c42c573f4a82099749d"},
        {"antenna_number":4, "report_location": "54f97c42c573f4a82099749d"}
	]
}
```

If you want to clear all report locations then you should send this JSON message:
```
{
	"report_location": null,
	"antenna_report_locations": []
}
```

Normally you would only have a filled `report_location` ór a filled `antenna_report_locations` field. If you supply both then items will become visible on multiple locations at the same time.

### Update remote senses
You can change a remote sense by sending a PUT request to the spot directly (api/spots/`id`). 5 remote sense are supported at this moment. bs1 to bs5 (where bs is short for brain sense). The remote sense are described in an object. Please note that any other key added to this object will be ignored at spot level.

You only need to add the senses that you want to change. I.e. updating the first three senses is done by sending the following object: 
```
{
	"senses_request": {
		"bs1":24,
		"bs2":1,
		"bs3":0
	}
}
```

The object is cleared as soon as the request is forwarded to the spot. The normal `senses` field will contain the current status of the senses.

Presences
---------

An item can be detected by multiple locations, if the signal is strong enough.  It's a 'deeper' more detailed view than you get when you would stick to the items and moves. In that case the server determines the most likely location of the item, even if it's present on multiple locations.

An item can be detected by multiple locatons at the same time. This is logical if you know that the used technolgies all have a big range. Your items may be picked up by multiple devices at the same moment. A new presence resource is automatically created when one of the defined locations says that an item is detected. A presence is deleted when it has not been seen for n seconds. Where n is the hold time in seconds (you may configure this parameters on your SmartSpot). So the presence resource exactly tells you where your items are beeing detected at this very moment. In this resource we present all this information to you. If you just only want to know where something is excatly located then we have good news: we already did the hard work for you. The localisation service does a best fit and determines where your item is. The calculated location is directly saved within the [items](#items) resource. You don't need to query this resource in that situation. This resource reveals more of what is happening inside the system. For some use cases this is really usefull.

Presence are deleted when their hold time expires, or in other words: when they have not been seen for a some time. This is an important difference to the first API version that we had. You can use the [events resource](#events) to query all events that took place in the system. Including create, update and delete events for presences. So you can always reconstruct the presences that where available at some time.

The presence contains these fields:

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `item` | [reference](README.md#reference) | Reference to the item that was detected. |
| `location` | [reference](README.md#reference) | Reference to the locaton that this item was seen on. |
| `proximity` | string | Strongest proximity of all 'child' presences, see next paragraph. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | Created time, when was the first hit received for this presence? |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last edit in this presence resource? |

We add an estimated proximity to every presence. This is a rough estimate on the distance from the item to the receiver. 3 possible values are returned:

1. 'far': the item is detected, but the received signal is weak. In most cases this means that the item is far away, but it also might indicate that you have interference or a seriously low battery.
2. 'near': the item is detected with an average signal strength.
3. 'immediate': the item is detected with a very strong signal. It must be very close to your antenna.

The returned value depends on the configured signal levels. It's possible to adjust these levels to your situation, please refer to the detailed Intellifi spot documentation if you would like to do this.

Subscriptions
-------------

Most resources contain actual state. The items for example are always available and can show you where an item is seen. If you have your own backend that keeps state then you are probably only intrested in the changes to our data. We call these events. Internally we make every change by sending events to internal services. We make these events available in several ways. If you create subscriptions in this collection then two things can happen:
1. Applying events are saved into the events resource.
2. Applying events are immediatly uploaded to an external HTTP end-point (webhooks).

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `topic_filter` | string | MQTT filter that is applied to all events. Allows you to select the events. |
| `description` | string | Add some notes if you like. |
| `database_hold_time_h` | value | The number of hours that this event is kept in the database. Only use larger numbers if you know what you are doing. A couple of hours is enough for most use cases. |
| `target_url` | string | This is a valid url to an external service that all applicable events are pushed to (webhook). Configure to null if you don't wish to use this (default). |
| `target_retry` | string | Set to true if you want our server to retry if target_url is not giving back a 2xx success code. |
| `verify_target_certificate` | string | Set to true if you want to force that the end-point is having a valid TLS certificate. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource created? |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change on this resource? |

Changes to subscriptions are applied automatically. The EventMemorizer service and webhook-svc are required to use this functionality (v2.0.0 and higher).

Events
------

The events resource keeps a copy of all relevant events that occured. This is an exact copy of the events that are available on the message bus. Please note that lots of events are flowing through the system. The history of events is kept for a limited time. If you would like to retreive all events then you should consider connecting to our message bus through one of the available [push technologies](https://github.com/intellifi-nl/doc-push).

Every event is envelopped in an JSON object with the following fields:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `topic` | object | A message is always accompanied by a topic string. In the brain we use a fixed format for this string. The topic string is parsed and the individual fields are shown in the object that is placed into this field. 
| `payload` | object | An object that contains the used encoding and the actual payload (if any). We will try to get this in line with the websockets output. Possible encodings: 'json', 'utf8' or 'base64'. We might add 'null', for now it's just an empty utf8 string if nothing was send.  |
| `time_event` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When did this event actually took place on the device? This is the device's own timestamp. Could be different due to buffering and clock differences. |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this event resource created at the server? |
| `time_expire` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When is this event going to removed from the database? |

The topic object is always filled with these properties:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `resource_type` | string | One of the defined [resources](#resource). Is also written in its plural form. I.e. 'spots', 'items'. |
| `resource` | [reference](README.md#reference) | Reference to one of the resources. Please note that it's an event from the past, the resource may not exist anymore. |
| `action` | string | Indicates the kind of event that was executed. In most cases it's a verb. I.e. 'connected', 'created' etc.
| `arguments` | object | Extra arguments may be added to a topic string, it depends on the `resource_type` and the `action` what extra arguments are added.

You may also query on these value by using a dot(.) in the url. More background information on the [topic format](https://github.com/intellifi-nl/doc-push/blob/master/mqtt_topics.md#format) can be found in the push documentation.

Be carefull with the given `time_create` and `time_device` fields. Intellifi Spots can buffer events in case of a network loss (a very small number for the moment). If you are traversing over the resource then you should look at `time_create`, eventual old events that pop up will just be in your result. If you are actually doing something with the event then you should look at `time_device`!

An event is never changed (can't be by definition!), so we don't offer a `time_update` field on this resource.

Services
--------

Brain background processes are called services. Every service keept it's own resource inside the services collection. It's a central place that allows you to check versions and tweak technical settings. You can post a JSON object to `config_request`, the service will change the setting when the time is there.

| Field | Type | Description | 
| ----- | ---- | ----------- |
| `url` | string | Url to the individual resource. |
| `id` | [ObjectId](http://docs.mongodb.org/manual/reference/object-id/) | Unique identifier for resource. |
| `name` | string | Human readable name for the service. |
| `version` | string | Current running version of the service. |
| `config` | object | JSON object with possible settings. Refer to individual service documentation for a good overview. |
| `config_request` | object | You may post a new object to this field in order to request a change in the service configuration. It shall be applied automatically (if you have choosen valid values and the service is running). |
| `restart_request` | bool | You may set this to true in order to request a restart for the service. |
| `boot_count` | value | Is increased with 1 when the service starts. Is never cleared (unless the database is adjusted). |
| `time_created` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was this resource created? |
| `time_updated` | [8601 string](http://en.wikipedia.org/wiki/ISO_8601) | When was the last change on this resource? |

Ideas
------

We are working constantly on new improvements and features. Some of the new forseen features are already mentioned here as a sneak preview.

* Paths, allows you to define a path through several locatons. Would create a passing object for items that match the exact path.
* Passings, would allow you to see certain movements of items through the system (as defined in paths). Basically this would be an advanced filter. Which of my items have moved from a to b and to c?
