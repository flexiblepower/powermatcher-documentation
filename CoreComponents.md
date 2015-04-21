This section gives an overview and to the point explanation of all the Core components. Most of them were discussed in the previous sections. In this section you might find some additional classes that make up the core and are as valuable for a proper working.

----------------------------

### Package: [[net.powermatcher.core|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core]]



**BaseAgent:**
Base implementation of an Agent. It provides basic functionality required in each Agent. Implements the ObservableAgent interface to make sure the instance can send AgentEvents to AgentObservers.

**BaseAgentEndpoint:** generic functionality of any Agent that implements a MatcherEndpoint interface to handle BidUpdate and PriceUpdate messages.

**BaseMatcherEndpoint:** generic functionality of any Agent that implements a MatcherEndpoint interface to handle incoming BidUpdates and outgoing PriceUpdate messages.

--------------------------

### Package: [[net.powermatcher.core.auctioneer|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/auctioneer]]

**Auctioneer:**
The Auctioneer balances and defines the market.

* The Auctioneer defines DataObject MarketBasis 
* The Auctioneer calculates the Equilibrium price based on the Aggregated Bid.
* The Auctioneer communicates the Equilibrium price to the Agents down the hierarchy in the form of priceUpdate messages.

**ObjectiveAuctioneer:**
The ObjectiveAuctioneer extends the normal Auctioneer and can now receive bids from the ObjectiveAgent that manipulates the cluster to pursue a certain Objective.

* The ObjectiveAuctioneer defines MarketBasis 
* The ObjectiveAuctioneer informs the ObjectiveAgent of the Aggregated Bid so it can determine its required action.
* The ObjectiveAuctioneer calculates the Equilibrium price based on the Aggregated Bid and Objective Bid.
* The ObjectiveAuctioneer communicates the Equilibrium price to the agents down the hierarchy in the form of priceUpdate messages.
 
See [[previous section|PowerMatcher-core]] for a detailed description of the bid aggregation process.

---------------------------

### Package: [[net.powermatcher.core.bidcache|https://github.com/flexiblepower/powermatcher/tree/development/net.powermatcher.core/src/net/powermatcher/core/bidcache]]

**BidCache:**
The BidCache keeps track of all bids that make up an aggregated bid, bids can be added and removed explicitly. The contents of the BidCache are constantly changing with bids added and removed through time.The Bid cache is fully thread-safe. 

**AggregatedBid:**
The AggregatedBid is the latest reflection of all BidUpdates aggregated into a single aggregated bid. The AggregatedBid extends the DataObject ArrayBid, see [[Data Objects|DataObjects]]. The extension of ArrayBid means that an aggregatedBid also keeps a list of Bids that were included in that aggregatedBid. 

----------------------------

### [[Package:net.powermatcher.core.concentrator|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/concentrator]]

**Concentrator:**
* The Concentrator aggregates Bids and forwards a single Bid.
* The Concentrator will receive price updates from the Auctioneer and forward them to its connected agents.

**PeakShavingConcentrator:**
* The PeakShavingConcentrator aggregates Bids and forwards a single Bid.
* The PeakShavingConcentrator informs an external service of the Aggregated Bid so it can determine its required action.
* The PeakShavingConcentrator will receive price updates from the Auctioneer
* The PeakShavingConcentrator will manipulate and forward the price based on a floor and ceiling constraints received from the external service.

-----------------------------

### [[Package:net.powermatcher.core.connectivity|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/connectivity]]

**BaseAgentEndpointProxy:**
This is the base implementation for remote agents. This is the "receiving end" of a remote communication pair.

**BaseAgentEndpointProxy:**
This is the base implementation for remote agents. This is the "sending end" of a remote communication pair.

------------------------

### [[Package:net.powermatcher.core.monitoring|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/monitoring]]

**LogRecord:**
An abstract class that contains the basic information needed to log an AgentEvent.

**AgentEventLogger:**
The basic class to store incoming AgentEvents. Subclasses of this abstract class implements their specific logging method in the dumpLogs() method.

**CSVLogger:**
An implementation of AgentEventLogger where the AgentEvents are logged to a comma separated file.

**AgentEventType:**
An Enum containing all existing AgentEvent implementations. This enum is used by
AgentEventLogger to be able to select an AgentEventType in the Apache Felix config admin.

**BaseObserver:**
Base class used to create an AgentObserver. The AgentObserver searches for ObservableAgent services and adds itself.
 
ObservableAgent services are able to call the update method of AgentObserver with AgentEvent events.

**BidLogRecord:**
An implementation of LogRecord that stores a BidEvent.

**PeakShavingLogRecord:**
An implementation of LogRecord that stores a PeakShavingEvent.

**PriceUpdateLogRecord:**
An implementation of LogRecord that stores a PriceUpdateEvent.

**WhitelistLogRecord:**
An implementation of LogRecord that stores a WhitelistLogRecord.

---------------------

### [[package:net.powermatcher.core.objectiveagent|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/objectiveagent]]

**BaseObjectiveAgent:**
The base implementation of a BaseObjectiveAgent. Each objective agent will require the interface of ObjectiveEndpoint with notifyPriceUpdate(Price) and handleAggregateBid(Bid).

-----------------------------

### [[Package:net.powermatcher.core.sessions|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/sessions]]

**SessionImpl:**
This class is the base implementation of the Session object.

It is responsible to hold serveral properties like sessionId, agentId, matcherId. There are two important methods. UpdateBid() and updatePrice(). UpdateBid() is called by Concentrator and Device Agents. UpdatePrice() is called by the Auctioneer and Concentrator.

**SessionManager:**
This class represents a SessionManager component which will store the active Sessions between an AgentEndpoint and a MatcherEndpoint object.

It is responsible for connecting and disconnecting an Auctioneer, Concentrator and agents. In activeSessions the Session will be stored. The SessionManager will connect a MatcherEndpoint to an agent and an AgentEndpoint with a MatcherEndpoint.

---------------------------

### [[Package:net.powermatcher.core.time|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.core/src/net/powermatcher/core/time]]

**LoggingScheduler:**
The base implementation of ScheduledExecutorService. It is used by all Powermatcher instances that have to perform periodic tasks.

**SystemTimeService:**
The base implementation of the TimeService interface. The "current time" in this class is the current time, returned by the System class.

------------------------------

Continue to [[PowerMatcher API|PowerMatcher-API]].