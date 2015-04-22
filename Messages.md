# Messages

Prices and Bids are clean Objects; however when they are sent to another Agent they become a message. Hence we have new Objects that represent Prices and Bids when they have become messages: `PriceUpdate` and `BidUpdate`.

##PriceUpdate

[PriceUpdate](https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.api/src/net/powermatcher/api/messages/PriceUpdate.java) is an immutable data object that stores a Price coupled with its bidNumber. 

```
    private final Price price;
    private final int bidNumber;
```

PriceUpdates are the messages sent between Agents and make sure a Price stays linked with a bidNumber. A Price and bidNumber are coupled to solve oscillation behaviour of a cluster, read more in [Oscillation](Oscillation) to uderstand the workings.

##BidUpdate
[BidUpdate](https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.api/src/net/powermatcher/api/messages/BidUpdate.java) is an immutable data object that stores a Bid and its bidNumber.

```
public class BidUpdate {
    private final Bid bid;
    private final int bidNumber;
```
The bidNumber is a Atomic Integer that is attached to a BidUpdate the moment it is published as a message. This happens in [BaseAgentEndpoint](https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.core/src/net/powermatcher/core/BaseAgentEndpoint.java)