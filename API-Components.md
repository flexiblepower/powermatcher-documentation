This section gives an overview and to the point explanation of all the API components. Most of them were discussed in the previous section. In this section you will find some additional classes that make up the API and are as valuable for a proper working.

------------------------------

### Package: [[net.powermatcher.api|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.api]]

**Agent:** Defines the interface with the basic functionality needed to act as an agent in a Powermatcher cluster.

**AgentEndpoint:** Defines the interface for classes that can receive a PriceUpdate. An AgentEndpoint can be linked with zero or one MatcherEndpoint instances. These are linked through a Session.

**MatcherEndpoint:** Defines the interface for classes that can receive a Bid. A MatcherEndpoint can be linked with zero or more AgentEndpoint instances. These are linked through a Session.

**ObjectiveEndpoint** Defines the interface for Objective Agents. Objective Agents are objects that can manipulate the Market in a Powermatcher cluster by sending a modified Bid.

**Session:** Defines the interface for a link between an AgentEndpoint with a MatcherEndpoint in a Powermatcher cluster.

--------------------------------

### [[Package: net.powermatcher.api.data|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.api/src/net/powermatcher/api/data]]

**MarketBasis:** Defines the Market: maximum, minimum Price etc.

**Bid:** Abstract of a BidObject

**ArrayBid:** Extends Bid and uses an demandArray to store a bidcurve.

**PointBid:** Extends Bid and uses pricePoints to store a bidcurve.

**Price:** The Price determined by the Auctioneer.

**PricePoint:** A X-Y coordinate that represents a cornerpoint on a bidcurve.

**PriceStep:** The index of a demandArray.

-------------------
### [[Package: net.powermatcher.api.messages|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.api/src/net/powermatcher/api/messages]]

**PriceUpdate:** The Object that ties a Price to a bidNumber.

**BidUpdate:** The Object that ties a Bid to a bidNumber. 

------------------

### [[Package: net.powermatcher.api.monitoring|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.api/src/net/powermatcher/api/monitoring]]

**ObservableAgent:** defines the interface with the basic functionality needed to be able to be observed by an AgentObserver.

**AgentObserver:** defines the interface with the basic functionality needed to observe an ObservableAgent and receive AgentEvents

---------------------------

#### [[Package: net.powermatcher.api.monitoring.events|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.api/src/net/powermatcher/api/monitoring/events]]

**AgentEvent:** This immutable abstract data object defines the basis for an event. AgentEvent is an event worthy of notifying another object of. Classes that have implemented the ObservableAgent interface can send an AgentEvent to every AgentObserver that observes this class.

**BidEvent:** A BidEvent extends AgentEvent to include information specically about a Bid.

**PriceUpdateEvent:** A PriceUpdateEvent extends AgentEvent to include information specifically about a PriceUpdate.

**IncomingBidEvent:** An IncomingBidEvent is sent when a MatcherEndpoint receives a new Bid.

**OutgoingBidEvent:** An OutgoingBidEvent is sent when an Agent sends a new Bid.

**IncomingPriceUpdateEvent:** An IncomingPriceUpdateEvent is sent when an AgentEndpoint receives a new PriceUpdate.

**OutgoingPriceUpdateEvent:** An OutgoingPriceUpdateEvent is sent when an Agent sends a PriceUpdate.


