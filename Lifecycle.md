# Lifecycle of Agents

Agents have a lifecycle that consists out of multiple parts. For the first part, OSGi Declarative Services is used to create and activate the component. This can either be a singleton object, but usually the factory pattern is used, such that for each configuration that is created, a new component is created (see TODO for a tutorial on this).

The component definition for an agent should look something like this:

```
@Component(designateFactory = ExampleAgent.Config.class,
           provide = { ObservableAgent.class, AgentEndpoint.class })
public class ExampleAgent extends BaseAgentEndpoint {
    public static interface Config {
        @Meta.AD(deflt = "concentrator")
        String desiredParentId();

        @Meta.AD(deflt = "freezer")
        String agentId();

        // Other configuration parameters
    }

    @Activate
    public void activate(Map<String, Object> properties) {
        config = Configurable.createConfigurable(Config.class, properties);
        activate(config.agentId(), config.desiredParentId());
        // Handle other configuration parameters
    }

    @Override
    public void setContext(FlexiblePowerContext context) {
        super.setContext(context);
        // Here you can use the context to schedule a recurring job, for example:
        scheduledFuture = context.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                doBidUpdate();
            }
        }, Measure.valueOf(0, SI.SECOND), Measure.valueOf(config.bidUpdateRate(), SI.SECOND));
    }

    // Here you can place your own logic for creating bids and handling price updates

    void doBidUpdate() {
        // TODO create new bid and call publishBid()
    }

    @Override
    public void handlePriceUpdate(PriceUpdate priceUpdate) {
        // Do something with the new price
    }
}
```

In this example, multiple instances of this agent can be created, because the `designateFactory` option of the component annotation is used. So for each configuration (as defined by the `Config` interface) a new instance of this agent is created. Also each agent is registered in the service registry as an `AgentEndpoint` and an `ObservableAgent` (see the `provide` option). This means that after activation this agent can be found by the PowerMatcher runtime.

The PowerMatcher runtime will do 2 things after it detects a new `AgentEndpoint` or `MatcherEndpoint`:

 - It will always call the `setContext` method to give the agent its `FlexiblePowerContext` instance. This object can be used to get the system time (which could be simulated!) and schedule jobs in the future (similar to having a `ScheduledExecutorService`).
 - If the agent implements the `AgentEndpoint` and it has found the `MatcherEndpoint` that has a matching `desiredParentId`, it will start up a `Session` between the two. For the `AgentEndpoint` this will result in a call to the `connectToMatcher` method (already implemented in the `BaseAgentEndpoint`) to give the reference to the session. This will in turn `configure` the agent with its `MarketBasis` and `clusterId`.

Only after the context has been set and the agent has been configured (either through connecting a session for a device agent or a concentrator, or by activation for the auctioneer), the agent is connected and can be used normally. This can be checked through the `Agent.isConnected()` method.

Here is a sequence diagram that describes this lifecycle:

![Agent Lifecycle](http://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgQWdlbnRFbmRwb2ludCBsaWZlY3ljbGUKCk9TR2kgLT4AGwY6IDw8Y3JlYXRlPj4AChBhY3RpdmF0ZShwcm9wZXJ0aWVzKQoAUgUAERRjb25maWd1cmF0aW9uKQBdCU9TR2k6IHJlZ2lzdGVyU2VydmljZShhZ2VudAAbClBvd2VyTWF0Y2hlciBSdW50aW1lOiBhZGQAgUAFACYIABIUAIE-C3NldENvbnRleHQoRmxleGlibGUAUwUADgcpCm5vdGUgb3ZlcgCCFgYsAGIVLCAAgQEHCiAgIFdoZW4gdGhlIGNvcnJlc3BvbmRpbmcgbQCBIwdpcyBmb3VuZCBhbmQKICAAJQUAEQtjb25uZWN0ZWQgdG8AQAZsdXN0ZXIKZW5kIG5vdGUAgTkZAIF6FgCDIgYgc2Vzc2lvbgCBdhkAgkEHOgB4CFRvAIJABgAxBwCCJyEALgkAgwoHAC8KAIIcJQogICBOb3cAgjIFAIQnBmlzIGFsc28AgXYaICAgaXNDAIIlCXdpbGwgcmV0dXJuIHRydWUgZnJvbSBub3cgb24AgjIKAIQeHnJlbW92ZQCEOQYAgygHAINNNE5vdyBhbnkAgmwIIHJlZ2FyAIN_BXRoaXMAhAAJAIEtBWJlIGRpcwCDbAkAg0QiAIZ6BwCERwcAhyMIRAA5CwCDEiIAg2QJAIZNBQAmHgCDBToAgUwMAIMIBgCDGCdmYWxzAIMqFg&s=modern-blue)
