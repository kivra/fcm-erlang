fcm-erlang
=======

This software provides an Erlang client for [`Firebase Cloud Messaging`](https://firebase.google.com/docs/cloud-messaging).

This supports both Android & iOS notifications.

The project is heavily inspired from [pdincau/gcm-erlang](https://github.com/pdincau/gcm-erlang). I would like to thank Paolo for the same.

### What you can do with fcm-erlang:

Using `fcm-erlang` you can:

1. start several `gen_servers` representing different `FCM applications` defined by different `FCM API keys`
2. send notification messages to Mobile devices registered to your specific application and registered to `FCM` using a specific `registration id`

`fcm-erlang` does only provide support for JSON messages.

### How to compile:

To compile

    $ rebar3 compile
    
To generate release

    $ rebar3 release
    
### How to add fcm as a dependency

In your rebar.config:

    {deps , [
        {fcm, ".*", {git, "https://github.com/softwarejoint/fcm-erlang.git", {tag, "1.0.1"}}}
    ]}.

### How to run the application:

`rebar3 release` will create a release under `_build/default/rel/fcm/` directory.

    $ cd _build/default/rel/fcm/
    $ ./bin/fcm console

Another way to run it:

    $ rebar3 shell --apps fcm

### How to start/stop different gen_servers under application fcm-erlang (one for each FCM application):

While `fcm-erlang` is running you can start several supervised gen_servers, one
for each FCM application. Every gen_server is defined by an atom used
internally for registration and by a `FCM API key`.

    3> fcm:start(foo, "myapikey").
    {ok,<0.60.0>}
    4> fcm:start(bar, "myotherapikey").
    {ok,<0.65.0>}
    5> fcm:start(baz, "mylastapikey").
    {ok,<0.79.0>}

You can stop a `gen_server` representing a FCM Application using:

    6> fcm:stop(foo).

### How to send a FCM message using from a specific FCM application:

At any time you can send a FCM message to one or more mobile devices by calling:

    7> fcm:push(RegisteredName, RegIds, Message).
    7> fcm:sync_push(RegisteredName, RegIds, Message).
    7> fcm:push(RegisteredName, RegId, Message).
    7> fcm:sync_push(RegisteredName, RegId, Message).

Where
 
	* `RegistereName` is the atom used during registration
	* `RegId` is Registration Id specified as Erlang binary (e.g., `<<"APA91bHun4MxP5egoKMwt2KZFBaFUH-1RYqx...">>`)
	* `RegIds` is a list (max 1000 elements) of `RegId`
        * `Message` is an Erlang Map or TupleList representing the data you want to send to the device.

In order to understand `Message` payload see [Message Syntax](https://firebase.google.com/docs/cloud-messaging/http-server-ref#send-downstream).

The JSON message is built using `jsx` in the module `fcm.erl` and in the end will have the following form:

    {
      "registration_ids" : ["APA91bHun4MxP5egoKMwt2KZFBaFUH-1RYqx..."],
      "data" : {
        "message" : "a message"
      },
      "time_to_live" : 3600,
      "collapse_key" : "your_update"
    }

You can send this message using:

    8> fcm:push(RegisteredName, RegIds, [{<<"data">>, [
    8>     {<<"message">>, <<"a message">>}
    8> ]}, {<<"time_to_live">>,3600}, {<<"collapse_key">>,<<"your_update">>}]).

or simply:

    8> fcm:push(RegisteredName, RegIds, [{<<"data">>, [
    8>     {<<"message">>, <<"a message">>}
    8> ]}]).

`fcm-erlang` will push the message for you to `Firebase Cloud Messaging` servers and will parse the JSON provided as result.

In order to understand errors see: [Interpreting an error response](https://firebase.google.com/docs/cloud-messaging/http-server-ref#error-codes).
