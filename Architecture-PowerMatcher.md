# Architecture PowerMatcher

In the [Introduction](Readme.md) we explained that the PowerMatcher creates a virtual market on which devices can place bids for consumption or production... Awesome! But how does this work?, well the underlying structure of The PowerMatcher is **Agent Technology**.

-------------------------------
By strictly defining the language or protocol that Agents must talk to each other we can place a lot of smartness in each Agent and thereby creating an overall very complex system. Think for example of a flock of birds, they follow three simple rules:

* Separation - avoid crowding neighbors (short range repulsion)
* Alignment - steer towards average heading of neighbors
* Cohesion - steer towards average position of neighbors (long range attraction)

With these three simple rules that each agent must comply with to participate in the system, the flock moves in an extremely realistic way, creating complex motion and interaction that would be extremely hard to create otherwise.

Our rules for participating in the system are:

* communicate Bids
* communicate Prices

Each Agent can strive for different goals as long as they communicate the PowerMatcher language/protocol that defines our system.

# The Agents

We distinguish at least 4 functional agents in a hierarchical structure:

* Auctioneer Agent
* Concentrator Agent
* Device Agent
* Objective Agent

![Example of a PowerMatcher cluster](http://flexiblepower.github.io/images/site/powermatcher.png)

## The Device Agent

In the PowerMatcher a device (washing machine, windmill, battery etc...) is represented by a device agent. It is therefore connected to both the PowerMatcher and to the physical device. **The device agent sends bids and receives prices from the PowerMatcher, by acting on the market it also influences the market. The device agent sends new setpoints (based on market prices) to the device and receives information on its current state (that can result in new bids).**

The device agent can be made even more advanced by including end user wishes/comfort levels or additional external input such as weather or energy price forecasts. By taking into account this extra information the device can determine for itself what would be the best time to consume…. or produce energy and consequently adapt his bid.

On one side the device agent is connected to the the PowerMatcher market, on the other side to the physical device.

To steer and read out the machine, a “physical” connection has to be made with the device. This could be done by mapping the agent directly to the IO or the device API. 

Alternatively PowerMatcher can also be run on the [EF-Pi technology](http://fpai-ci.sensorlab.tno.nl/builds/fpai-documentation/v14.10/html/) as an intermediate layer; this only requires a mapping of the device agent to the abstraction layer of EF-Pi. EF-Pi takes care of the actual physical connection with the device (see EF-Pi section). As a result the PowerMatcher device agent does not have to think about the type of machine, brand, or model number. The reader is referred to the EF-Pi section for a more detailed explanation of EF-Pi technology.

## The Auctioneer Agent

The Auctioneer always stands on top of the hierarchy and aggregates all bids received from "child" Agents. The Auctioneer **determines the equilibrium** of the cluster and returns the price to each Agent....this market price will be the incentive for devices to start producing or consuming....more...or...less! The price will balance the system so the consumption or production level will be different for every device.

## The Concentrator Agent

The Concentrator agent performs three functions. 

The first function of the concentrator is the aggregation of bids received from child agents. Children could be other concentrator agents or other device agents. The Concentrator concentrates, or **aggregates**, all these bids and publishes a single aggregated bid upward in the hierarchy thereby **reducing communications in the system making it very scalable**.

The second function entails the virtual representation of concepts or physical entities in the real world. A concentrator can bundle every smart device in a single household and therefore represent that household. A concentrator can also represent an electric cable, transformer or for instance a neighborhood. 

Lastly, the concentrator can be used to manipulate the children agents that are present in a sub cluster. Affecting a single concentrator allows for localized manipulation. The first idea of local manipulation that has been implemented in the PowerMatcher is the capability of local congestion management; by making the concentrator more intelligent it can peak shave loads and make sure that the net output of sub cluster stays within certain power limits.

## The Objective Agent

The objective agent gives a cluster its purpose. The objective agent **interfaces to the business logic** behind the specific application.  When the objective agent is absent, the goal of the cluster is to balance itself, i.e., it strives for an equal supply and demand within the cluster itself. Depending on the specific application, the goal of the cluster might be different. If the cluster has to operate as a virtual power plant, for example, it needs to follow a certain externally provided setpoint schedule. Such an externally imposed objective can be realized by implementing an objective agent. Other couplings could be with the energy trading market, external optimization algorithms etc.

# Other Demand Response Systems

Important to understand is that The PowerMatcher always optimizes the current state based on the latest information. 

Other Demand Response systems often work based on some central optimization algorithm that executes three steps: Forecasting, Scheduling and Real Time Coordination. In the PowerMatcher core we focus on Real Time Coordination, we believe that Forecasting and Scheduling can be performed locally by each individual agent; which results in much more complex dynamics.
