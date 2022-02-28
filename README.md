# Intempt - Nodejs SDK

[Intempt](https://intempt.com/?utm_campaign=sdk&utm_medium=docs&utm_source=github) Customer Data Platform built on Open Data Stack for the needs of SAAS and eCommerce companies focusing to grow their customer LTV.
This is a library to facilitate tracking of anonymous and logged-in user traffic on your Node.js server.

## Contents:

* [1](https://github.com/intempt/intempt-nodejs#installation-and-initialization) - Installation and Initialization 
* [2](https://github.com/intempt/intempt-nodejs#identifying-visitors) - How to identify a user
* [3](https://github.com/intempt/intempt-nodejs#recording-events) - Recording Events
* [4](https://github.com/intempt/intempt-nodejs#create-custom-collections) - Create Custom Collections
* [5](https://github.com/intempt/intempt-nodejs#batching) - Batching
* [6](https://github.com/intempt/intempt-nodejs#get-apis) - Get Apis

## Installation and Initialization

Install our SDK which is also an npm package by running 
```Javascript
    npm install intempt-nodejs-source
```

To update to the latest version, run

```Javascript
    npm install intempt-nodejs-source@latest
```

[Login](https://app.intempt.com) to Intempt and obtain your snippet from the [sources](https://app.intempt.com/sources) page:

Add the following code to your nodejs server, inside the `index.js` file:

```javascript
    const Intempt_sdk = require("intempt-nodejs-source");
    const Intempt = new Intempt_sdk({
        organization: "<organization-name>",
        sourceId: "<source-id>",
        apiKey: "<api-key>"
    })
```

You can get an automatically generated version of this code, with your account specific variables filled in, from the sources page.

It also accepts a set of options which tell the SDK what to do. These options are optional and not required. These options include;

`flushAt`: This has a value of number. This has a default value of `10`. This determines how many items can be stored in memory at a time. For example, if you have `flushAt` set at `1`, this means we can only have one event at a time in our batch memory. 

`flushInterval`: This has a value of number. This has a default value of `10000ms`. This determines how long our memory should hold data before `flushing` to Intempt Servers.. For example, if you have `flushInterval` set at `10000ms`, this means we can only store events for as long as `10 seconds` in our batch memory. 

# Identifying Users

Our SDK provides an exported function to automatically track users. To track a user, all you have to do is to call the function below

```javascript
	Intempt.identify({profileId: profileId, user_identifier: user_identifier, account_identifier: account_identifier})
```

`profileId` : This is a profile Identifier which is used to identify users. ```Note```: In your server, it is advisable to use the Id of your users as profileId. That way, it is always unique and you don't need to have a randomizer or genrator.

`user_identifier` : User identifier helps to identify users across all sources in your organization. This could be the user's email and so on. Eg. `example@example.com`


# Recording Events

Our server-side API allows creation of any event type you might desire to be tracked for your page or application.

Custom collections allow you to bring custom data into your organization.

# Create Custom Collections
After creating a nodejs source, switch tabs to the schema and you should see a `profile` and `consents` collections already created. To start tracking custom events for our organization, after creating a [source](https://app.intempt.com/sources) you intend to use on your server, navigate to schema then use the schema Builder to create a collection with your personalized fields. Make sure to add a profileId field and set a profile identifier as one of its properties. [See intempt docs for keys and identifiers](https://dev.intempt.com/#keys-and-identifiers) to find out why.

Set up other custom field properties that match the data you plan to track.

Our SDK by default has a few exported functions for certain event tracking such as `Identify`, `trackConsents` . You can log a custom event by calling Intempt's recordEvent function, passing in a set of key-value pairs.


```javascript
    Intempt.recordEvent('COLLECTION_NAME', event);
```

The COLLECTION_NAME refers to an event type. For example, if you wanted to track every time your server received a purchase request, you might call that collection "purchase". Then, when a purchase happens, you call track and pass in the purchase details.

### Example

```javascript
	Intempt.recordEvent("purchase", {
		 "items": [{"item name": "item 1","price": 20}, {"item name": "item 2","price": 15}],
		 "totalprice": 35,
		 "ispaid": true,
		 "timestamp": new Date().getTime(),
		 "fixed.name": "John Smith",
		 "fixed.age": 28,
		 "intempt.visit.trackcharge": 35
	})
```

In the example above, we have a custom collection with name of `purchase`, then we have an object of key-value pairs. The Object passed into represent the fields we created while creating the collection. 

`NB`: Data type sent should match data type set when creating the collection. For instance, We are sending a field `timestamp` which has a Data type of `number`, Sending data with a type of `string` instead of `number` will make this custom collection not get tracked

# Tracking Consents

Save user consents and apply them to data flow. Make sure you have created consents purposes in your organization's settings



### Supported Regulation Types - GDPR & CCPA;

Consents are tracked using Intempt's exported function ;

```javascript
	Intempt.trackConsents(consents)
```

`Intempt.trackConsents()` expects consents as an array of Objects inorder to be tracked successfully. 
Example: 

```Javascript
	Intempt.trackConsents([
		{
         "regulation": "GDPR",
         "purpose": "advertising",
         "consented": true
        },
		{
         "regulation": "CCPA",
         "purpose": "advertising",
         "consented": false
        },
	])

```

Consents can contain as much as possible depending on your use case.

# Batching

Our libraries are built to support high-performance environments. That means it is safe to use our Node library on a web server that’s serving hundreds of requests per second.

Every method you call does not result in an HTTP request, but is queued in memory instead. Messages are then flushed in batch in the background, which allows for much faster operation.

By default, our library will flush:
The very first time it gets a message.
Every 10 messages `(controlled by options.flushAt)`.
If 10 seconds has passed since the last flush (controlled by options.flushInterval)

There is a maximum of 500KB per batch request and 32KB per call.
If you don’t want to batch messages, you can turn batching off by setting the flushAt option to 1, like so:

```Javascript
    const Intempt_sdk = require("intempt-nodejs-source");
    const Intempt = new Intempt_sdk({
        organization: "<organization-name>",
        sourceId: "<source-id>",
        apiKey: "<api-key>"
    }, {flushAt: 1})
```

You can also flush on demand. For example, at the end of your program, you need to flush to make sure that nothing is left in the queue. To do that, call the flush method:

```javascript
    Intempt.flush();
```

`Note`: For `flushAt` to work, you need to add one more event greater than the value of `flushAt`. For Instance, `{flushAt: 10}`. This means that for the flush to be triggered, you have to add `11 events` to memory. When this happens, it flushes the memory and adds the new event to memory making total events in memory `1`.

# GET APIS
Below is a detailed list of functions our SDK makes available to be used in your server.
# EVENTS

`Intempt.getEventsByUser(payload)`: Fetches all Events by user.

`Intempt.getEventsByUserWithEventFilterQuery(payload)`: Fetches all Events by user and Event Filters which include eventId or eventName.

payload required: user: `String`, type: `String`, eventIds: `Array`, eventNames: `Array`

### When to pass eventIds or eventNames:
When you are passing type with a value of eventId, then eventIds becomes `required` and thesame for passing type with a value of eventName, eventNames become `required`.

`Intempt.getEventsbyMasterId(masterid)`: Fetches all Events by masterId


`Intempt.getEventsByProfileId(profileId, config)`; Fetches all Events by profileId

`Intempt.getEventsBySourceId(sourceId, profile, config)`: Fetches all Events by sourceId

payload required: sourceId: `String`, profile: `String`, config; `Object` {`singleProfile`: true}; 

Config is not required as singleProfile will be set to false by default.



# SEGMENTS

`Intempt.getSegments()`: Fetches all segments for your organization.

`Intempt.getSegmentsByProfileId(profileid)`: Fetches all segments by profileId.

`Intempt.getSegmentsBymasterId(masterid)`: Fetches all segments by masterId.

`Intempt.getSegmentsByUser(user)`: Fetches all segments by user.

`Intempt.getSegmentsByUserWithSegmentIdFilter(payload)`: Fetches all segments by user and segment Ids.

payload required: user: `String`, segmentIds: `Array` [`'id1'`, `'id2'`]

`Intempt.getSegmentsByMasterIdWithSegmentNameFilter(payload)`: Fetches all segments by user and segment Names

payload required: masterId: `String`, segmentNames: `Array` [`'segment1'`, `'segment2'`]

`Intempt.getSegmentsBySourceId(sourceId, profile)`: Fetches all segments by sourceId

payload required: sourceId: `String`, profile: `String`


# METRICS
`Intempt.getMetricValuesByMasterId(masterId)`: Fetches Metric Values by masterId

`Intempt.getMetricValuesByUser(user)`: Fetches Metric Values by user

`Intempt.getMetricValuesByUserWithMetricFilters(payload)`: Fetches all Metric Values by user and Metric Filters which include metricIds or metricNames.

payload required: user: `String`, type: `String`, metricIds: `Array`, metricNames: `Array`

### When to pass eventIds or eventNames:
When you are passing type with a value of metricName, then metricNames becomes `required` and thesame for passing type with a value of metricId, metricIds become `required`.

`Intempt.getMetricByProfileId(profileId)`: Fetches Metric Values by profileId

`Intempt.getMetricValuesBySourceId(payload)`: Fetches Metric Values by sourceId.
payload is an `Object`. payload required: sourceId: `String`, profile: `String`


# USERS
`Intempt.getUsersByMasterId(masterId)`: Fetches all users by masterId

`Intempt.getUsersByUser(user)`: Fetches all users by user

`Intempt.getUsersByUserWithMetricIdFilter(payload)`: : Fetches all users by user and metric Id Filter

payload required: user: `String`, metricIds: `Array` [`'id1'`, `'id2'`]

`Intempt.getUsersWithIncludedFilter(payload)`; Fetches all users with included filters

payload required: metricNames: `Array`, segmentNames: `Array`, attributeNames: `Array`, eventNames: `Array`, included: `Array`

### When to pass metricNames or attributeNames or segmentNames or eventNames:

before any of these can be passed, it must be added in the `included` Array. For example: if included is: (included: ['Attributes','Segments', 'Events', 'Metrics']), then we pass ({attributeNames: ['attribute1'], segmentNames: ['segment1'], eventNames: ['event1'], metricNames: ['metricName']}).

`Intempt.getUsersByProfileId(profileId)`: Fetches all users by profileId.

`Intempt.getUsersBySourceId(payload)`: Fetches all users by sourceId. 
payload is an `Object`. payload required - sourceId: `String`, profile: `String`
{ sourceId, profile }

`getUsers()`: Fetches all users without any filter.


# USER-ATTRIBUTES

`Intempt.getAttributeValuesByMasterId(masterId)`: Fetches all user-attributes by masterId

`Intempt.getAttributeValuesByUser(user)`: Fetches all user-attributes by user

`Intempt.getAttributeValuesByUserWithAttributeFilters(payload)`: Fetches all user-attributes by user and Attributes Filters which include attributeId or attributeName.

payload required: user: `String`, type: `String`, attributeIds: `Array`, attributeNames: `Array`

### When to pass attributeIds or attributeNames:
When you are passing type with a value of attributeId, then attributeIds becomes `required` and thesame for passing type with a value of attributeName, attributeNames become `required`.

`Intempt.getAttributesByProfileId(profileId)`: Fetches all user-attributes by profileId

`Intempt.getAttributeValuesBySourceId(payload)`: Fetches all user-attributes by sourceId
payload is an `Object`. payload required: sourceId: `String`, profile: `String`

