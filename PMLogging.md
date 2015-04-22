# Overview

This section explains the logging of `Events`. The slf4j logging is not handled here.

## Observing Events

Powermatcher implements the [Observer pattern](http://en.wikipedia.org/wiki/Observer_pattern). Two interfaces that make this possible. 

* `ObservableAgent` : classes that implements this interface, can be "observed". This means that an `ObservableAgent` can send out an `AgentEvent` to all its observers. `AgentObserver` can be added and removed by calling `addObserver` and `removeObserver` respectively.
* `AgentObserver` : This class "observes" `ObservableAgents`. It has an `update` method that can be called by the `ObservableAgent` when they want to send an `AgentEvent`.

## Logging events

Now for the part we're interested in: logging these `AgentEvents`. Powermatcher has an [AgentEventLogger](https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.monitoring.csv/src/net/powermatcher/monitoring/csv/AgentEventLogger.java) base class with the basic functionality of a logger. It stores the events and makes sure the events are logged after the given time interval has passed. The most important method any Logger that extends (aside from the OSGi code), is the `dumpLogs()` method. In this method holds the code for your specific type of logging.

