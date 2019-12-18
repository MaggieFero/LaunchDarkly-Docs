---
title: "PHP SDK Reference"
excerpt: ""
---
This reference guide documents all of the methods available in our PHP SDK, and explains in detail how these methods work. If you want to dig even deeper, our SDKs are open source-- head to our [PHP SDK repository](https://github.com/launchdarkly/php-server-sdk) on GitHub. Additionally you can clone and run a [sample application](https://github.com/launchdarkly/hello-php) using this SDK.
[block:callout]
{
  "type": "info",
  "title": "Requirements",
  "body": "PHP 5.5 or higher."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Getting started"
}
[/block]
Building on top of our [Quickstart](doc:getting-started) guide, the following steps will get you started with using the LaunchDarkly SDK in your PHP application.

The first step is to install [Composer](https://getcomposer.org/) and the LaunchDarkly SDK as a dependency in your application. Refer to the [SDK releases page](https://github.com/launchdarkly/php-server-sdk/releases) to identify the latest version if you want to depend on a specific version.
[block:code]
{
  "codes": [
    {
      "code": "php composer.phar require launchdarkly/server-sdk\n\n# Note that in earlier versions, this was \"launchdarkly/launchdarkly-php\"",
      "language": "shell"
    }
  ]
}
[/block]
Then require Composer's autoloader.
[block:code]
{
  "codes": [
    {
      "code": "require 'vendor/autoload.php';",
      "language": "php"
    }
  ]
}
[/block]
Once the SDK is installed, you'll want to create a single, shared instance of `LDClient`. You should specify your SDK key here so that your application will be authorized to connect to LaunchDarkly and for your application and environment.
[block:code]
{
  "codes": [
    {
      "code": "$client = new LaunchDarkly\\LDClient(\"YOUR_SDK_KEY\");",
      "language": "php"
    }
  ]
}
[/block]
Using `$client`, you can check which variation a particular user should receive for a given feature flag.
[block:code]
{
  "codes": [
    {
      "code": "$user = new LaunchDarkly\\LDUser(\"user@test.com\");\n\nif ($client->variation(\"your.flag.key\", $user)) {\n  # application code to show the feature\n} else {\n  # the code to run if the feature is off\n}",
      "language": "php"
    }
  ]
}
[/block]
That's all!

You may remember from our [Quickstart](doc:getting-started) guide that users should be sure to shut down the LaunchDarkly client on application termination. This step does not exist in PHP because the PHP SDK does not maintain long-lived network connections nor an event queue.
[block:api-header]
{
  "type": "basic",
  "title": "Fetching Flags"
}
[/block]
There are two distinct methods of integrating LaunchDarkly in a PHP environment.  

- [Guzzle Cache Middleware](https://github.com/Kevinrob/guzzle-cache-middleware) to request and cache HTTP responses in an in-memory array (default)
- [ld-relay](https://github.com/launchdarkly/ld-relay) to retrieve and store flags in Redis (recommended)

We strongly suggest using the ld-relay.  Per-flag caching mode (Guzzle) is only intended for low-throughput environments.
[block:api-header]
{
  "title": "Using Guzzle"
}
[/block]
Require Guzzle as a dependency:
[block:code]
{
  "codes": [
    {
      "code": "php composer.phar require \"guzzlehttp/guzzle:6.2.1\"\nphp composer.phar require \"kevinrob/guzzle-cache-middleware:1.4.1\"",
      "language": "shell"
    }
  ]
}
[/block]
It will then be used as the default way of fetching flags.

With Guzzle, you could persist your cache somewhere other than the default in-memory store, like Memcached or Redis.  

You could then specify your cache when initializing the client with the cache option.
[block:code]
{
  "codes": [
    {
      "code": "$client = new LaunchDarkly\\LDClient(\"YOUR_SDK_KEY\", array(\"cache\" => $cacheStorage));\n",
      "language": "php"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Using LD-Relay"
}
[/block]
Setup [ld-relay](https://github.com/launchdarkly/ld-relay) in [daemon-mode](https://github.com/launchdarkly/ld-relay#redis-storage-and-daemon-mode) with Redis

Require Predis as a dependency:

    php composer.phar require "predis/predis:1.0.*"

Create the LDClient with the Redis feature requester as an option:

    $client = new LaunchDarkly\LDClient("your_sdk_key", ['feature_requester_class' => 'LaunchDarkly\LDDFeatureRequester', 'redis_host' => 'your.redis.host', 'redis_port' => 6379]);
[block:callout]
{
  "type": "danger",
  "title": "Caching and PHP",
  "body": "PHP's *shared-nothing* architecture means that the out-of-the-box in-memory HTTP cache used by our SDK won't cache feature flags across requests.\n\nIn production, we strongly recommend the ld-relay for PHP."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Customizing your client"
}
[/block]
In an earlier example, we passed a `cache` option as an array to the client constructor. There are a few additional options you can set in this array. Here's an example:
[block:code]
{
  "codes": [
    {
      "code": "$client = new LaunchDarkly\\LDClient(\"YOUR_SDK_KEY\", array(\"cache\" => $cacheStorage, \"connect_timeout\" => 3));",
      "language": "php"
    }
  ]
}
[/block]
We've set the client connect timeout to 3 seconds in addition to providing a custom cache storage provider. The complete list of customizable parameters is as follows:

* `connect_timeout`: Must be a number, in seconds, and controls the connection timeout to LaunchDarkly
* `timeout`: Must be a number, in seconds, and controls the end-to-end request timeout to LaunchDarkly
* `capacity`: Must be a number, and controls the maximum size of the event buffer. LaunchDarkly sends events asynchronously, and buffers them for efficiency. 
* `base_uri`: Base URI of the LaunchDarkly API. Defaults to `https://app.launchdarkly.com`.
* `events_uri`: Base URI for sending events to LaunchDarkly. Defaults to 'https://events.launchdarkly.com'
* `cache`: Sets the feature flag cache to be used by the client. Defaults to an in-memory cache.
* `send_events`: Boolean value which enables sending events back to LaunchDarkly. Defaults to `true`.
* `logger`: Configures a logger for warnings and errors generated by the SDK. Defaults to a `Monolog\Logger` sending all messages to the php error_log.
* `offline`: Whether the client should be initialized in offline mode. In offline mode, default values are returned for all flags and no remote network requests are made. Defaults to `false`.
* `feature_requester`: An optional `LaunchDarkly\FeatureRequester` instance.
* `feature_requester_class`: An optional class implementing `LaunchDarkly\FeatureRequester`, if `feature_requester` is not specified. Defaults to a `GuzzleFeatureRequester`.
* `event_publisher`: An optional `LaunchDarkly\EventPublisher` instance.
* `event_publisher_class`: An optional class implementing `LaunchDarkly\EventPublisher`, if `event_publisher` is not specified. Defaults to `CurlEventPublisher`.
* `allAttributesPrivate`: Whether all user attributes (except the user key) should be marked as [private user attributes](https://docs.launchdarkly.com/docs/private-user-attributes) and not sent to LaunchDarkly. Defaults to `false`.
* `privateAttributeNames`: Must be a list of strings. The names of user attributes that should be marked as [private user attributes](https://docs.launchdarkly.com/docs/private-user-attributes), and not sent to LaunchDarkly. Defaults to an empty array.
[block:callout]
{
  "type": "warning",
  "title": "Sending events in PHP",
  "body": "The LaunchDarkly SDK sends data back to our server to record events from Track and Variation calls. On our other platforms, this data is sent asynchronously, so that it adds no latency to serving web pages. \n\nOnce again, PHP's *shared-nothing* architecture makes this a bit difficult. By default, LaunchDarkly forks an external process that executes `curl` to send this data. In practice we've found that this is the most reliable way to send data without introducing latency to page load times. \n\nIf your server does not have `curl` installed, or has other restrictions that make it impossible to invoke `curl` as an external process, you may need to implement a custom `EventProcessor` to send events to LaunchDarkly."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Users"
}
[/block]
Feature flag targeting and rollouts are all determined by the *user* you pass to your `Variation` calls. In our PHP SDK, we use a [builder pattern](http://en.wikipedia.org/wiki/Builder_pattern) to make it easy to construct users. Here's an example:
[block:code]
{
  "codes": [
    {
      "code": "$user = (new LDUserBuilder(\"aa0ceb\"))->firstName(\"Ernestina\")->lastName(\"Evans\")->email(\"ernestina@example.com\")->custom([\"groups\" => array(\"Google\",\"Microsoft\")])->build();\n\n",
      "language": "php"
    }
  ]
}
[/block]
Let's walk through this snippet. The first argument to the builder is the user's key-- in this case we've used the hash `"aa0ceb"`. **The user key is the only mandatory user attribute**. The key should also uniquely identify each user. You can use a primary key, an e-mail address, or a hash, as long as the same user always has the same key. We recommend using a hash if possible.

All of the other attributes (like `firstName`, `email`, and the `custom` attributes) are optional. The attributes you specify will automatically appear on our dashboard, meaning that you can start segmenting and targeting users with these attributes. 

Besides the `key`, LaunchDarkly supports the following attributes at the "top level". Remember, all of these are optional:

* `ip`: Must be an IP address.
* `firstName`: Must be a string. If you provide a first name, you can search for users on the Users page by name.
* `lastName`: Must be a string. If you provide a last name, you can search for users on the Users page by name.
* `country`: Must be a string representing the country associated with the user. 
* `email`: Must be a string representing the user's e-mail address. If an `avatar` URL is not provided, we'll use [Gravatar](http://en.gravatar.com/) to try to display an avatar for the user on the Users page.
* `avatar`: Must be an absolute URL to an avatar image for the user. 
* `name`: Must be a string. You can search for users on the User page by name
* `anonymous`: Must be a boolean. See the section below on anonymous users for more details.

In addition to these, you can pass us any of your own user data by passing `custom` attributes, like the `groups` attribute in the example above. 
[block:callout]
{
  "type": "info",
  "title": "A note on types",
  "body": "Most of our built-in attributes (like names and e-mail addresses) expect string values. Custom attributes values can be strings, booleans (like true or false), numbers, or lists of strings, booleans or numbers. \n\nIf you enter a custom value on our dashboard that looks like a number or a boolean, it'll be interpreted that way. The PHP SDK is strongly typed, so be aware of this distinction."
}
[/block]
Custom attributes are one of the most powerful features of LaunchDarkly. They let you target users according to any data that you want to send to us-- organizations, groups, account plans-- anything you pass to us becomes available instantly on our dashboard.
[block:api-header]
{
  "title": "Private user attributes"
}
[/block]
You can optionally configure the PHP SDK to treat some or all user attributes as [private user attributes](https://docs.launchdarkly.com/v2.0/docs/private-user-attributes).  Private user attributes can be used for targeting purposes, but are removed from the user data sent back to LaunchDarkly.

In the PHP SDK there are two ways to define private attributes for the **entire** LaunchDarkly client:

*In the LaunchDarkly `config`, you can set `allAttributesPrivate` to `true`. If this is enabled, all user attributes (except the key) for all users are removed before the user is sent to LaunchDarkly.
*In the LaunchDarkly `config` object, you can define a list of `privateAttributeNames`. If any user has a custom or built-in attribute named in this list, it will be removed before the user is sent to LaunchDarkly.

You can also mark attributes as private when building the user object itself by calling the equivalent "private" user builder method. For example: 
[block:code]
{
  "codes": [
    {
      "code": "$user = (new LDUserBuilder('aa0ceb'))\n  ->privateEmail('test@example.com')\n\t->build();",
      "language": "php"
    }
  ]
}
[/block]
When this user is sent back to LaunchDarkly, the `email` attribute will be removed.


[block:api-header]
{
  "type": "basic",
  "title": "Anonymous users"
}
[/block]
You can also distinguish logged-in users from anonymous users in the SDK, as follows:
[block:code]
{
  "codes": [
    {
      "code": "$user = (new LDUserBuilder(\"aa0ceb\"))->anonymous(true)->build();",
      "language": "php"
    }
  ]
}
[/block]
You will still need to generate a unique key for anonymous users-- session IDs or UUIDs work best for this. 

Anonymous users work just like regular users, except that they won't appear on your Users page in LaunchDarkly. You also can't search for anonymous users on your Features page, and you can't search or autocomplete by anonymous user keys. This is actually a good thing-- it keeps anonymous users from polluting your Users page!
[block:api-header]
{
  "type": "basic",
  "title": "Variation"
}
[/block]
The `variation` method determines which variation of a feature flag a user receives.
[block:code]
{
  "codes": [
    {
      "code": "$value = $client->variation($key, $user, false);",
      "language": "php"
    }
  ]
}
[/block]
`variation` calls take the feature flag key, an `LDUser`, and a default value. 

The default value will only be returned if an error is encountered-- for example, if the feature flag key doesn't exist or the user doesn't have a key specified. 

The `variation` call will automatically create a user in LaunchDarkly if a user with that user key doesn't exist already. There's no need to create users ahead of time (but if you do need to, take a look at Identify).
[block:api-header]
{
  "type": "basic",
  "title": "Track"
}
[/block]
The `track` method allows you to record actions your users take on your site. This lets you record events that take place on your server. In LaunchDarkly, you can tie these events to goals in A/B tests. Here's a simple example:
[block:code]
{
  "codes": [
    {
      "code": "$client->track(\"your-goal-key\", user);",
      "language": "php"
    }
  ]
}
[/block]
You can also attach custom data (anything that can be marshaled to JSON) to your event by passing an extra parameter to `track`: 
[block:code]
{
  "codes": [
    {
      "code": "$client->track(\"Completed purchase\", user, [\"price\" => 320]);",
      "language": "php"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Identify"
}
[/block]
The `identify` creates or updates users on LaunchDarkly, making them available for targeting and autocomplete on the dashboard. In most cases, you won't need to call `identify`-- the `variation` call will automatically create users on the dashboard for you. `identify` can be useful if you want to pre-populate your dashboard before launching any features. 
[block:code]
{
  "codes": [
    {
      "code": "$client->identify(user);",
      "language": "php"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "All flags"
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Note that unlike variation and identify calls, allFlagsState does not send events to LaunchDarkly. Thus, users are not created or updated in the LaunchDarkly dashboard.",
  "title": "Creating users"
}
[/block]
The `allFlagsState` method captures the state of all feature flag keys with regard to a specific user. This includes their values, as well as other metadata.

This method can be useful for passing feature flags to your front-end. In particular, it can be used to provide bootstrap flag settings for our [JavaScript SDK](doc:js-sdk-reference).
[block:code]
{
  "codes": [
    {
      "code": "$state  = $client->allFlagsState($user);",
      "language": "php"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Secure mode hash"
}
[/block]
The `secureModeHash` method computes an HMAC signature of a user signed with the client's SDK key. If you're using our [JavaScript SDK](doc:js-sdk-reference) for client-side flags, this method generates the signature you need for secure mode.
[block:code]
{
  "codes": [
    {
      "code": "$client->secureModeHash(user);",
      "language": "text"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Offline mode"
}
[/block]
In some situations, you might want to stop making remote calls to LaunchDarkly and fall back to default values for your feature flags. For example, if your software is both cloud-hosted and distributed to customers to run on premise, it might make sense to fall back to defaults when running on premise. `setOffline` lets you do this easily. 
[block:code]
{
  "codes": [
    {
      "code": "$client = new LaunchDarkly\\LDClient(\"YOUR_SDK_KEY\", array(\"offline\" => true));\n$client->variation(\"any.feature.flag\", user, false); // will always return the default value (false)\n\n",
      "language": "php"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Logging"
}
[/block]
The PHP SDK uses [Monolog](https://github.com/Seldaek/monolog). All loggers are namespaced under `LaunchDarkly`. A custom logger may be passed to the SDK via the configurable `logger` property:
[block:code]
{
  "codes": [
    {
      "code": "$client = new LaunchDarkly\\LDClient(\"YOUR_SDK_KEY\", array(\"logger\" => new Logger(\"LaunchDarkly\", [new ErrorLogHandler(0, Logger::DEBUG)])));\n\n",
      "language": "php",
      "name": "Logging"
    }
  ]
}
[/block]
Be aware of two considerations when enabling the DEBUG log level:

1. Debug-level logs can be very verbose. It is not recommended that you turn on debug logging in high-volume environments.
2. Potentially sensitive information is logged including LaunchDarkly users created by you in your usage of this SDK.