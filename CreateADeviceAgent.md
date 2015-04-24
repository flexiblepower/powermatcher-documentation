# Create a Device Agent

In this part of the tutorial, you will learn how to create your own Device Agent. The device we will be creating is a WindTurbine. It will be a simulated WindTurbine which is not actually connected to a physical device. Therefore we won't receive actual Flexibility Information [see Device Agent Bids](Bids.md), we will simulate this output.

# Creating the basic class

In eclipse, right click on the `net.powermatcher.examples` package in the `net.powermatcher.examples` project. Select new -> Class. Enter `WindTurbine` in the Name field and click on Finish.

## Inheritance

PowerMatcher has a `BaseAgentEndpoint` that does generic handling of sending BidUpdates and receiving PriceUpdates, so we'll extend from it. A device agent also has to implement the `AgentEndpoint` interface. 

It should look like this:

```
public class WindTurbine extends BaseAgentEndpoint implements AgentEndpoint {

 }

```
## Adding/overriding methods

To function as a proper Device Agent you will need to add two important methods; one for sending BidUpdates and one for handling PriceUpdates. The method `handlePriceUpdate()` already has an implementation in the `BaseAgentEndpoint` and needs to be Overridden.
Don't forget to call the `super.handlePriceUpdate()`

```
    void doBidUpdate() {
    }

	
    @Override
    public synchronized void handlePriceUpdate(PriceUpdate priceUpdate) {
	    super.handlePriceUpdate(priceUpdate);
    }
```

## Adding Fields
    
Now let's create some fields that we can use to simulate our Device:

```
	//Important fields to assign the Agent's Logger, Agent configuration and scheduler
	
    private static final Logger LOGGER = LoggerFactory.getLogger(PVPanelAgent.class);

	private Config config;
	
    private ScheduledFuture<?> scheduledFuture;
	
	

	//Variables that will be needed for simulating our WindTurbine bid
	
	private static Random generator = new Random();
	
    private double minimumDemand;

    private double maximumDemand;
```

## The actual Business Logic

Now we get to the actual functionality of our `WindTurbine`: sending `Bids`and receiving `Prices`.

### Sending bids

To send Bids, we will implement `doBidUpdate()`. The basic functionality of this method is to create a `Bid` and send that bid to a `MatcherEndpoint` through the `Session`. For the purposes of this demo, we will be sending randomly generated bids. But in a real Device Agent; new Bids will often be the result due a change in the state of the Device...so if the Wind will start blowing harder it will want to offer more Power to the market: hence the maximumDemand will probably coupled to the Power Output of the Windmill in some way.....see [Device Agent Bids](Bids.md).  

1. Check the status if the Agent is still connected
2. Use the generator to create a random demand, based on the `minimumDemand` and the  `maximumDemand`
2. Create a flat Bid using the `flatDemand()`. More complex shapes can be constructed using the Builder pattern, see the [Freezer Example Device Agent](https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.examples/src/net/powermatcher/examples/Freezer.java).
3. Publish the Bid using `publishBid()` it will send the Bid to its `MatcherEndpoint` through the `Session`. 

```
    void doBidUpdate() {
        AgentEndpoint.Status currentStatus = getStatus();
        if (currentStatus.isConnected()) {
            double demand = minimumDemand + (maximumDemand - minimumDemand) * generator.nextDouble();
            publishBid(Bid.flatDemand(currentStatus.getMarketBasis(), demand));
        }
    }
```

### Receiving prices

A Price is received from the MatcherEndpoint/Parent Agent w the `handlePriceUpdate(PriceUpdate priceUpdate)` method. In the demo we won't be doing anything with the incoming Price.  


```
   @Override
    public synchronized void handlePriceUpdate(PriceUpdate priceUpdate) {
        super.handlePriceUpdate(priceUpdate);
        // Nothing to control for a WindTurbine
```

# OSGI

Last up are the OSGI configuration, annotations and methods.

## The Config Interface

We will start by creating an inner interface called Config. It doesn't have to be called Config, but PowerMatcher uses that name. This class will be the `Service Factory object` and OSGi will use it to create new instances with the given Parameters.

A `WindTurbine` instance needs 5 properties: 

* `agentId` : The id of this windTurbine.
* `desiredParentId ` : The id of the desired parent.
* `bidUpdateRate` : The number of seconds between bid updates.
* `minimumDemand` : The mimimum value of the random demand.
* `maximumDemand` : The maximum value the random demand.

We will use the OSGi `@Meta.AD` annotation to provide a default value for both fields and a description for these values.

```
public static interface Config {
    @Meta.AD(deflt = "windturbine")
    String agentId();
	
	@Meta.AD(deflt = "concentrator")
    String desiredParentId();

    @Meta.AD(deflt = "5", description = "Number of seconds between bid updates")
    long bidUpdateRate();

    @Meta.AD(deflt = "-1", description = "The mimimum value of the random demand.")
    double minimumDemand();

    @Meta.AD(deflt = "-2000000", description = "The maximum value the random demand.")
    double maximumDemand();
}
```

Now the `WindTurbine` class itself can be annotated and `Config` can be configured to be the `Service Factory object`

```
@Component(designateFactory = WindTurbine.Config.class, immediate = true, provide = { ObservableAgent.class, AgentEndpoint.class })
public class WindTurbine extends BaseAgentEndpoint implements AgentEndpoint {

...

```

## LifeCycle Management

To create managed instances, OSGi needs 2 things: an `activate` method and a `deactivate` method.

### Activate

We will start by creating an configurable instance of our `Config` interface to be able to access the configuration parameters.

Finally, this method has to be annotated with `@Activate` 

```
    @Activate
    public void activate(Map<String, Object> properties) {
        config = Configurable.createConfigurable(Config.class, properties);
        init(config.agentId(), config.desiredParentId());

        minimumDemand = config.minimumDemand();
        maximumDemand = config.maximumDemand();

        LOGGER.info("Agent [{}], activated", config.agentId());
    }
```

And the PowerMatcher Runtime has to be able to set the `Context`. The Context is supplied by the FlexiblePower Base API.
It takes care of the clock and allows the Agent to schedule a `Runnable`. In this example the Agent will schedule a doBidUpdate() with a delay of 0 zero seconds and at a fixed interval equal to bidUpdateRate().
See [FlexiblePower Base API]() for more scheduling options

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

That's it! You just created a simple Device Agent. You can now fire up OSGI like we did in the [SimplePMCluster](SimplePMCluster.md) section and add a `WindTurbine` to the mix. 

If the `WindTurbine `does not show up in your config admin, you will probably have to rebuild the project with the bundle file of the net.powermatcher.examples project.

```
package net.powermatcher.examples;

import java.util.Map;
import java.util.Random;
import java.util.concurrent.ScheduledFuture;

import javax.measure.Measure;
import javax.measure.unit.SI;

import net.powermatcher.api.AgentEndpoint;
import net.powermatcher.api.data.Bid;
import net.powermatcher.api.data.PointBid;
import net.powermatcher.api.data.Price;
import net.powermatcher.api.data.PricePoint;
import net.powermatcher.api.messages.PriceUpdate;
import net.powermatcher.api.monitoring.ObservableAgent;
import net.powermatcher.core.BaseAgentEndpoint;

import org.flexiblepower.context.FlexiblePowerContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import aQute.bnd.annotation.component.Activate;
import aQute.bnd.annotation.component.Component;
import aQute.bnd.annotation.component.Deactivate;
import aQute.bnd.annotation.metatype.Configurable;
import aQute.bnd.annotation.metatype.Meta;

/**
 * {@link PVPanelAgent} is a implementation of a {@link BaseAgentEndpoint}. It represents a dummy freezer.
 * {@link PVPanelAgent} creates a {@link PointBid} with random {@link PricePoint}s at a set interval. It does nothing
 * with the returned {@link Price}.
 * 
 * @author FAN
 * @version 2.0
 */
@Component(designateFactory = WindTurbine.Config.class,
           immediate = true,
           provide = { ObservableAgent.class, AgentEndpoint.class })
public class WindTurbine
    extends BaseAgentEndpoint
    implements AgentEndpoint {

    private static final Logger LOGGER = LoggerFactory.getLogger(WindTurbine.class);

    private Config config;

    public static interface Config {
        @Meta.AD(deflt = "windturbine", description = "The unique identifier of the agent")
        String agentId();

        @Meta.AD(deflt = "concentrator",
                 description = "The agent identifier of the parent matcher to which this agent should be connected")
        public String desiredParentId();

        @Meta.AD(deflt = "5", description = "Number of seconds between bid updates")
        long bidUpdateRate();

        @Meta.AD(deflt = "-1", description = "The mimimum value of the random demand.")
        double minimumDemand();

        @Meta.AD(deflt = "-2000000", description = "The maximum value the random demand.")
        double maximumDemand();
    }

    /**
     * A delayed result-bearing action that can be cancelled.
     */
    private ScheduledFuture<?> scheduledFuture;

    private static Random generator = new Random();

    /**
     * The mimimum value of the random demand.
     */
    private double minimumDemand;

    /**
     * The maximum value the random demand.
     */
    private double maximumDemand;

    /**
     * OSGi calls this method to activate a managed service.
     * 
     * @param properties
     *            the configuration properties
     */
    @Activate
    public void activate(Map<String, Object> properties) {
        config = Configurable.createConfigurable(Config.class, properties);
        init(config.agentId(), config.desiredParentId());

        minimumDemand = config.minimumDemand();
        maximumDemand = config.maximumDemand();

        LOGGER.info("Agent [{}], activated", config.agentId());
    }

    /**
     * OSGi calls this method to deactivate a managed service.
     */
    @Override
    @Deactivate
    public void deactivate() {
        super.deactivate();
        scheduledFuture.cancel(false);
        LOGGER.info("Agent [{}], deactivated", getAgentId());
    }

    /**
     * {@inheritDoc}
     */
    void doBidUpdate() {
        AgentEndpoint.Status currentStatus = getStatus();
        if (currentStatus.isConnected()) {
            double demand = minimumDemand + (maximumDemand - minimumDemand) * generator.nextDouble();
            publishBid(Bid.flatDemand(currentStatus.getMarketBasis(), demand));
        }
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public synchronized void handlePriceUpdate(PriceUpdate priceUpdate) {
        super.handlePriceUpdate(priceUpdate);
        // Nothing to control for a WindTurbine
    }

    /**
     * {@inheritDoc}
     */
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
}

```