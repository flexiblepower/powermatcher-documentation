# Creating a Device Agent

##Overview

In this part of the tutorial, you will learn how to create your own device agent. The device we will be creating is a wind turbine.

# Creating the basic class

In eclipse, right click on the `net.powermatcher.examples` package in the `net.powermatcher.examples` project. Select new -> Class. Enter `WindTurbine` in the Name field and click on Finish.

## Inheritance

Powermatcher has a `BaseAgentEndpoint` that does generic handling of sending BidUpdates and receiving PriceUpdates, so we'll extend from it. A device agent also has to implement the `AgentEndpoint` interface. Next, hover your mouse over WindTurbine and select `Add unimplemented methods`. Eclipse will provide all the methods that need to be implemented.

```
public class WindTurbine extends BaseAgentEndpoint implements AgentEndpoint {

 
    @Override
    public void setContext(FlexiblePowerContext context) {
        // TODO Auto-generated method stub
        
    }
   
    @Override
    public void connectToMatcher(Session session) {
        // TODO Auto-generated method stub
        
    }

    @Override
    public void matcherEndpointDisconnected(Session session) {
        // TODO Auto-generated method stub
        
    }

    @Override
    public void handlePriceUpdate(PriceUpdate priceUpdate) {
        // TODO Auto-generated method stub
        
    }
    

}


```
## Adding fields

The BaseAgentEndpoint provides a lot of fields, but we have to add the following ourselves:

*
*

    
Now that we have these fields, let's create some setters methods:

*
*

```
todo
```

## The actual Business Logic

Now we get to the actual functionality of our `WindTurbine`: sending `Bids`and receiving `Prices`.

### Sending bids

To send Bids, we will implements `doBidUpdate()`. The basic functionality of this method is to create a `Bid` and send that bid to a `MatcherEndpoint` through the `Session`. For the purposes of this demo, we will be sending randomly generated bids. But in a real Device Agent; new Bids will often be the result due a change in the state of the Device...so if the Wind will start blowing harder it will want to offer more Power to the market: hence the maximumDemand will probably coupled to the Power Output of the Windmill in some way.....see [[Device Agent Bids|Bids]].  

1. Use the generator to create a random demand, based on the `minimumDemand` and the  `maximumDemand`
2. Create a PointBid using the `Builder` pattern to construct the `Bid` out of two `PricePoints` (Price, Demand) (see [[Data Objects|DataObjects]]
3. Publish the Bid. `publishBid()` is defined in BaseAgentEndpoint and makes sure that the Bid gets a unique bidNumber, it sends an `OutgoingBidUpdateEvent` to all `Observers` of this `WindTurbine` instance, and send the Bid to its `MatcherEndpoint` through the `Session`. 

```
    void doBidUpdate() {
        if (isInitialized()) {
            double demand = minimumDemand + (maximumDemand - minimumDemand)
                            * generator.nextDouble();

            publishBid(new PointBid.Builder(getMarketBasis()).add(getMarketBasis().getMinimumPrice(), demand)
                                                             .add(getMarketBasis().getMaximumPrice(), minimumDemand)
                                                             .build());
        }
    }
```

### Receiving prices

A Price is updated by a MatcherEndpoint calling the `handlePriceUpdate(PriceUpdate priceUpdate)` method. Again, in the demo we won't be doing anything with the incoming Price. The `BaseAgentEndpoint` makes sure that an `IncomingPriceUpdateEvent` is sent to all `Observers` of this `WindTurbine` instance. 

The only action we will undertake is to check whether the bidNumber received in `PriceUpdate` is consistent with the bidNumber of our latest BidUpdate:

```
    @Override
    public void handlePriceUpdate(PriceUpdate priceUpdate) {
        LOGGER.debug("Received price update [{}], current bidNr = {}",
                     priceUpdate, getLastBidUpdate().getBidNumber());
    }
```

# OSGI

Last up are the OSGI configuration, annotations and methods.

## The Config interface

We will start by creating an inner interface called Config. (It doesn't have to be called Config, but Powermatcher uses that name.) This class will be the `Service Factory object` and OSGi will use it to create new instances with the given Parameters.

A `WindTurbine` instance needs 5 properties: 

* `desiredParentId ` : The id of the desired parent.
* `agentId` : The id of this windTurbine.
* `bidUpdateRate` : The number of seconds between bid updates.
* `minimumDemand` : The mimimum value of the random demand.
* `maximumDemand` : The maximum value the random demand.

We will use the OSGi `@Meta.AD` annotation to provide a default value for both fields and a description for these values.

```
public static interface Config {
    @Meta.AD(deflt = "concentrator")
    String desiredParentId();

    @Meta.AD(deflt = "windturbine")
    String agentId();

    @Meta.AD(deflt = "30", description = "Number of seconds between bid updates")
    long bidUpdateRate();

    @Meta.AD(deflt = "-700", description = "The mimimum value of the random demand.")
    double minimumDemand();

    @Meta.AD(deflt = "-600", description = "The maximum value the random demand.")
    double maximumDemand();
}
```

Now the class itself can be annotated and `Config` can be configured to be the `Service Factory object`

```
@Component(designateFactory = WindTurbine.Config.class, immediate = true, provide = { ObservableAgent.class, AgentEndpoint.class })
public class WindTurbine extends BaseAgentEndpoint implements AgentEndpoint {
```

## Lifecycle management

To create managed instances, OSGi needs 2 things: an `activate` method and a `deactivate` method.

### Activate

We will start by creating an configurable instance of our `Config` interface to be able to access the configuration parameters.

Finally, this method has to be annotated with `@Activate` 

```
    @Activate
    public void activate(Map<String, Object> properties) {
        config = Configurable.createConfigurable(Config.class, properties);
        activate(config.agentId(), config.desiredParentId());

        minimumDemand = config.minimumDemand();
        maximumDemand = config.maximumDemand();

        LOGGER.info("Agent [{}], activated", config.agentId());
    }
```

And the PowerMatcher Runtime has to be able to set the `Context`:

```
    @Override
    public void setContext(FlexiblePowerContext context) {
        super.setContext(context);
        scheduledFuture = context.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                doBidUpdate();
            }
        }, Measure.valueOf(0, SI.SECOND), Measure.valueOf(config.bidUpdateRate(), SI.SECOND));
    }
```

### Deactivate

This method disconnects the Session and cancels the scheduledFuture that is running the thread doing all the bid updates. Add some logging and you're done.
Similar to the `activate` method, this method will be annotated with `@Deactivate`.

```
    @Override
    @Deactivate
    public void deactivate() {
        super.deactivate();
        scheduledFuture.cancel(false);
        LOGGER.info("Agent [{}], deactivated", getAgentId());
    }
```

#Final class

That's it! You just created a simple Device Agent. You can now fire up OSGI like we did in the [SimplePMCluster](Simple PM cluster) section and add a `WindTurbine` to the mix. 

If the `WindTurbine `does not show up in your config admin, you will probably have to rebuild the project with the bundle file of the net.powermatcher.examples project.

```
todo
```