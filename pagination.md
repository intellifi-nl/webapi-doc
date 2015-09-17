Collection resource pagination
==============================

Our API offers different collection resources (items, locations etc). A collection resource contains 0..n resources where n could grow to pretty large numbers. It's a good thing to be able to request these resources in multiple smaller requests (pages). Let's investigate the default behaviour and your options to tune this pagination process to your needs.

Default
-------

We start limiting a collection when the number of contained resources is larger than 100. The newest resources are shown first. This should always allow you to do a quick check on collection resources in your browser.

The resources are always wrapped into an response object (unless you requested csv output). This object gives you important information about the request but also about the pagination. The most important field is the url_next, following it takes you 'through' the whole collection.

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/pagination.png)

Pleae note: newest resources are shown first, so the pagination is going from new to old. As a consequence you can completely read all resources. The url_next is set to null when you reach the end of the collection. Another consequence of this is that you won't receive new resources when you are stepping through the pages. You would need to start a new itteration.

Parameters
----------

You may increase or decrease the number of returned documents by adding `limit` to your query parameters. It would be usefull to increase `limit` when you have a high lattency between client and server. A high limit would decrease the number of roundtrips. At this moment you may set any limit you like, please don't use huge limits. It would cause high server loads and an unresponsive application. An example: limiting a resource to 5 resources: [http://brain.intellifi.nl/api/items?limit=5](http://brain.intellifi.nl/api/items?limit=5)

You can adjust the order by adding the `sort` parameter to your query parameters. You can sort on all fields in the resource, you may even sort on multiple fields. Just add them seperated by commas. The default order is ascending, you may make a order descending by adding a minus sign (-) in front of the field name. The default `sort` value is `-id`.

Off course you can combine the query parameters. Let's say that you want to request a top 10 of present items that have the highest number of moves: [http://brain.intellifi.nl?sort=-move_count&limit=10&is_present=true](http://brain.intellifi.nl?sort=-move_count&limit=10&is_present=true)

Please note that pagination only works when you sort on `id`. This can both ascending `sort=id` (the default `sort` value) and descending `sort=-id`. The url_next is always set to null when you sort on other fields.

Following
---------

If you sort ascending (add `sort=id` to your query parameter) then you paginate from old to new. This is a logical choice if you are creating an application that wants to 'follow' new resources in a collection. Especially the events or moves resource are very usefull to query this way. It allows your application to process events from old to new.

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/pagination-old-new.png)

Please note that the next_url is always filled! This is because newer resource always can be created at the end of your itteration. So make sure that your implementation does not end up in an endless loop when the `is_limited` field becomes false. In fact it should make it easier for you to create a background worker that polls for new events.
