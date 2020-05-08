---
layout: post
title: "Nameko Eventlog Dispatcher"
author: Julio Trigo
date: 2017-08-08 09:25:05 +0100
modified_date: 2019-03-21 00:00:00 +0000
last_modified_at: 2020-05-08 18:00:00 +0000
permalink: /articles/nameko-eventlog-dispatcher/
redirect_from: /posts/nameko-eventlog-dispatcher/
tags:
  - nameko
  - python
  - microservices
---

Here at [Sohonet](https://twitter.com/sohonet), we build our products backend following a microservices architecture.

Our services are built using [Nameko](https://github.com/nameko/nameko), a microservices framework for Python that *"lets service developers concentrate on application logic and encourages testability"*.

Nameko encourages the use of dependency injection and provides an easy way of adding *dependencies* to our services.<!--more-->

## Our need

There are many cases where we want to react to some of the events that happen in our applications. For instance, in our [ClearView Flex](https://www.sohonet.com/clearview-flex/) real time collaboration application, we want to log some data for some of our events and also be able to programatically query that data later on. Some examples of these events are: *streaming session generated*, *attendee notified*, *streaming started*, etc.

Nameko already has an `EventDispatcher` dependency provider that can be used to dispatch events. However, what we need is to be able to dispatch both custom event data and some **metadata** related to each one of those events.

Additionally, since what we need in this case is to log events data, we want to avoid having to handle each one of those event types individually.

## Our solution

In order to achieve that, we have built [`nameko-eventlog-dispatcher`](https://github.com/sohonetlabs/nameko-eventlog-dispatcher), a *"Nameko dependency provider that dispatches log data using Events (Pub-Sub)"*.

This dependency provider inherits from `EventDispatcher`, but enriches the event data with the metadata that we need. It also provides an ***auto capture*** mode, that allows us to log all the calls made to any of the entrypoints in our service.

### Dispatching event log data

In order to use the dependency, we need to include the `EventLogDispatcher` dependency provider in our service class and manually call it to dispatch an event:

```python
from nameko.rpc import rpc
from nameko_eventlog_dispatcher import EventLogDispatcher


class FooService:

    name = 'foo'

    eventlog_dispatcher = EventLogDispatcher()

    @rpc
    def foo_method(self):
        self.eventlog_dispatcher(
          'foo_event_type', {'value': 1}, metadata={'meta': 2}
        )
```

Calling `foo_method` dispatches an event from the `foo` service with `log_event` as the event type. However `foo_event_type` will be the event type included in the event metadata.

`event_type`, `event_data` (optional) and `metadata` (optional) can be provided as arguments. Both `event_data` and `metadata` must be dictionaries and contain JSON **serializable** data.

Then, any Nameko service will be able to handle this event.

```python
from nameko.events import event_handler


class BarService:

    name = 'bar'

    @event_handler('foo', 'log_event')
    def foo_log_event_handler(self, body):
        """`body` will contain the event log data."""
```

### Auto capture enabled

When ***auto capture*** is enabled, a Nameko event is automatically dispatched every time an entrypoint is fired.

We can achieve that by overriding the `worker_setup` method in the dependency provider, which is called before a service worker executes a task when an entrypoint is fired.

### Format of the event log data

This is an example of event data:

```python
{
  "service_name": "foo",
  "entrypoint_protocol": "Rpc",
  "entrypoint_name": "foo_method",
  "call_id": "foo.foo_method.d7e907ee-9425-48a6-84e6-89db19e3ce50",
  "call_stack": [
    "standalone_rpc_proxy.call.3f349ea4-ed3e-4a3b-93d0-a36fbf928ecb",
    "bla.bla_method.21d623b4-edc4-4232-9957-4fad72533b75",
    "foo.foo_method.d7e907ee-9425-48a6-84e6-89db19e3ce50"
  ],

  "event_type": "foo_event_type",  # the kind of event being dispatched: "session_created", "entrypoint_fired"...
  "timestamp": "2017-06-12T13:48:16+00:00",

  "meta": 2,  # extra information provided as "metadata"
  "data": {"value": 1}  # extra information provided as "event_data"
}
```

The `data` attribute will contain the event data provided when the event was dispatched. If the event was automatically dispatched (***auto capture***) then `data` will be empty.

If `metadata` was provided, then its elements will be included as top level attributes.

### Setup

An `EVENTLOG_DISPATCHER` element can be added (optional) to our Nameko configuration file to override some of the setup default values:

```yaml
## config.yaml

EVENTLOG_DISPATCHER:
  auto_capture: true  # enables auto capture mode
  entrypoints_to_exclude: []  # list of entrypoint names to exclude when auto capture mode is enabled
  event_type: log_event  # event type used to dispatch all the events
```

## Summary

The [`nameko-eventlog-dispatcher`](https://github.com/sohonetlabs/nameko-eventlog-dispatcher) library can be installed from PyPI using pip:

```shell
$ pip install nameko-eventlog-dispatcher
```

By overriding the existing `EventDispatcher` Nameko dependency provider, we get all its functionally for free, making it also easier to add our custom logic that both dispatches events and adds the metadata that we need.

We've also tried to limit the internal Nameko components that we use in our code to the minimum in order to reduce coupling and prevent incompatibilities with future Nameko releases if its internal implementation changes.

All the log events are dispatched using the same event type (`log_event`) and are grouped per service:

- Dispatching all the log events using the same *event type* makes it easier to add new events for our services. The original event type used when the event is dispatched is included in the `event_type` metadata attribute.
- Log events are individually handled per service, rather than grouping them for all our services. This allows us to have more granular control over what we log and when we start handling log events for a service. It creates different queues per service in `RabbitMQ` which could be beneficial in terms of performance, if the amount of logs varies significantly between services.

***2019-01-28 - Update:***

A new version of the library has been released: [`0.3.0`](https://pypi.org/project/nameko-eventlog-dispatcher/).

We've dropped support for previous [Nameko](https://github.com/nameko/nameko) versions: it only officially supports [`v2.11.0`](https://github.com/nameko/nameko/releases/tag/v2.11.0) now, which is the latest release on the `2.x` branch at the moment.

***2019-03-21 - Update:***

A new version of the library has been released: [`0.4.1`](https://pypi.org/project/nameko-eventlog-dispatcher/).

We've added support for older [Nameko](https://github.com/nameko/nameko) versions, including the latest one in the `2.x` series: `2.6`, `2.7`, `2.8`, `2.9`, `2.10`, `2.11`, `2.12`.
