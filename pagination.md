Collection resource pagination
==============================

Our API offers different collection resources (items, locations etc). A collection resource contains 0..n resources where n could grow to pretty large numbers. It's a good thing to be able to request these resources in multiple smaller requests (pages). Let's investigate the default behaviour and your options to tune this pagination process to your needs.

Normal behaviour
----------------

We start limiting a request response when the number of resources is larger than 100. The newest resources are always shown first. This should always allow you to do a quick check on collection resources in your browser.

The results are wrapped into an object (unless you requested csv output). This object gives you important information about the request but also about the pagination. The most important field is the `url_next`, following it takes you 'through' the whole collection. In this picture you see 8 resources (green boxes). In this example the server limits the number of results to a maximum of 3:

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/pagination.png)

* The collections show newest resources first, so the pagination is going from new to old.
* The `url_next` field is set to null when you reach the end of the collection.
* You will not receive new resources while stepping through the complete collection. You would need to start a new itteration.
* You could ignore the `url_next` and construct your own pagination, please do ** not ** do this. We could change the implementation of the pagination, if we do this then we also change the generated `url_next` fields. Loose coupling is a good thing (and also [good practice](https://en.wikipedia.org/wiki/HATEOAS)).

Ordering
--------
You can adjust the order by adding the `sort` parameter to your query parameters. You can sort on all fields in the resource, you may even sort on multiple fields. Just add them seperated by commas. The default order is ascending, you may make a order descending by adding a minus sign (-) in front of the field name. The default `sort` value is `-id`.

Please note that pagination only works when you sort on `id`. This can both ascending `sort=id` (the default `sort` value) and descending `sort=-id`. The `url_next` is always set to null when you sort on other fields.

Changing the limit
------------------
You may increase or decrease the number of returned documents by adding `limit` to your query parameters. I.e. requesting 5 maximal resources per page would be done by: [http://brain.intellifi.nl/api/items?limit=5](http://brain.intellifi.nl/api/items?limit=5) It would be usefull to increase `limit` when you have a high lattency between client and server. It would decrease the number of required roundtrips. At this moment you may set any `limit` value you like, please don't use huge values (>1000). It would cause high server loads and an unresponsive application. 

Off course you can combine the query parameters. Let's say that you want to request a top 10 of present items that have the highest number of moves: [http://brain.intellifi.nl/api/items?sort=-move_count&limit=10&is_present=true](http://brain.intellifi.nl/api/items?sort=-move_count&limit=10&is_present=true)

Following
---------

If you sort ascending (add `sort=id` to your query parameter) then pagination is going from old to new. This is a logical choice if you are creating an application that wants to 'follow' new resources in a collection. Especially the events or moves resource are very usefull to query this way. It allows your application to process events beginning to end. Here is an example image to illustrate this:

![](https://raw.githubusercontent.com/intellifi-nl/doc-webapi/master/pagination-old-new.png)

* Please note that the next_url is always filled! This is because newer resource always can be created at the end of your itteration. The `next_url` may return no results, you can just reuse the url until something is created. Please note that the `is_limited` becomes false when the number of results is smaller than `limit`.
* This is the advised way to retreive events from our API with a background worker (if you don't use one of our pushing technologies). Make sure that your implementation does not end up in an endless loop when you follow the `next_url`. Also make sure that your process remembers the last `next_url` when it's restarting for some reason. You probably don't want to reread already processed events.
