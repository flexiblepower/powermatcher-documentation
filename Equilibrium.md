Make sure you have read the [[Aggregation|Aggregation]] section before you continue. 

Assume that we now have a very small cluster with only two Device Agents: the Freezer and the Battery, and the Auctioneer. See Figure 1:

![Auctioneer Equilibrium](AuctioneerEquilibrium.png)

**Figure 1: A very small PowerMatcher Cluster.**

The Auctioneer receives an Aggregated Bid similar to the Bid in [[Aggregation|Aggregation]]:

![AggregatedBid](AggregatedBid1.png)

**Figure 2: The Aggregated Bid @ the Auctioneer.**

-----------------------------------------------
# Determining the Equilbrium Price

The Auctioneer performs one extra action after aggregation: to determine the optimal point of the system. 

Since the final aggregated Bid represents the **net power demand** as a function of the price, the price where the **net demand equals zero** is the point where the systems supplies as much power as is demanded.

Figure 3 shows this in a visual representation of an Aggregated Bid that consists of only two bids; one consumption bid by a Freezer and a production bid by a Battery.

![Equilibrium](Equilibrium1.png)

**Figure 3: Determining the Equilibrium price!**

The point where the aggregated bid passes through the X-axes/Price-axes: this is the internal price where the system is in balance. This is a method of the [[Auctioneer|https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.core/src/net/powermatcher/core/auctioneer/Auctioneer.java#L211]] 

After determining the Equilibrium price it is [[communicated down the PowerMatcher to each individual device|https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.core/src/net/powermatcher/core/auctioneer/Auctioneer.java#L198]]. Each device will start consuming or producing energy as ‘promised’ by its Bid. See Figure 4:

![NewSetPoint](NewSetPoint.png)

**Figure 4: Device Agents determine their New Setpoints based on the received Equilibrium Price.**

# Technical Implementation

Aggregation happens the same as discussed in the previous section.

The Auctioneer's `performUpdate()` is activated and calls `calculateIntersection()`

```
    @Override
    protected void performUpdate(AggregatedBid aggregatedBid) {
        Price price = aggregatedBid.calculateIntersection(0);
        publishPrice(price, aggregatedBid);
    }

```

The function `.calculateIntersection()` resides in the object ArrayBid. `AggregatedBid` extends ArrayBid but also stores all references to the Bids that have contributed to that `AggregatedBid`. An ArrayBidis explained in more detail in [[Data Objects|DataObjects]].

```
    @Override
    public Price calculateIntersection(double targetDemand) {
        int leftIx = 0, rightIx = demandArray.length - 1;

        // First test for a few special cases
        if (targetDemand > demandArray[leftIx]) {
            // If the target is higher than the maximum of the bid, return the minimum price
            return new Price(marketBasis, marketBasis.getMinimumPrice());
        } else if (targetDemand < demandArray[rightIx]) {
            // If the target is lower than the minimum of the bid, return the maximum price
            return new Price(marketBasis, marketBasis.getMaximumPrice());
        } else if (demandIsEqual(targetDemand, demandArray[leftIx])) {
            rightIx = leftIx;
        } else if (demandIsEqual(targetDemand, demandArray[rightIx])) {
            leftIx = rightIx;
        } else { // demand is between the limits of this bid, which can not be flat at this point
            // Go on while there is at least 1 point between the left and right index
            while (rightIx - leftIx > 1) {
                // Determine the middle between the 2 boundaries
                int middleIx = (leftIx + rightIx) / 2;
                double middleDemand = demandArray[middleIx];

                if (demandIsEqual(targetDemand, middleDemand)) {
                    // A point with the target demand is found, select this point
                    leftIx = rightIx = middleIx;
                } else if (middleDemand > targetDemand) {
                    // If the middle demand is bigger than the target demand, we set the left to the middle
                    leftIx = middleIx;
                } else { // middleDemand < targetDemand
                    // If the middle demand is smaller than the target demand, we set the right to the middle
                    rightIx = middleIx;
                }
            }
        }

        // If the left or right point matches the targetDemand, expand the range
        while (leftIx > 0 && demandIsEqual(targetDemand, demandArray[leftIx - 1])) {
            leftIx--;
        }
        while (rightIx < demandArray.length - 1 && demandIsEqual(targetDemand, demandArray[rightIx + 1])) {
            rightIx++;
        }

        return interpolate(leftIx, rightIx, targetDemand);
    }

```

The Price is then published to each connected Agent/Session which is handled in the [[BaseMatcherEndpoint|https://github.com/flexiblepower/powermatcher/blob/development/net.powermatcher.core/src/net/powermatcher/core/BaseMatcherEndpoint.java]] of the Auctioneer, see [[PowerMatcher API|PowerMatcher-API]] for more information:

```
    public void publishPrice(Price price, AggregatedBid aggregatedBid) {
        Map<String, Integer> references = aggregatedBid.getAgentBidReferences();

        for (Session session : sessions.values()) {
            Integer bidNumber = references.get(session.getAgentId());
            if (bidNumber != null) {
                PriceUpdate priceUpdate = new PriceUpdate(price, bidNumber);
                publishEvent(new OutgoingPriceUpdateEvent(session.getClusterId(),
                                                          getAgentId(),
                                                          session.getSessionId(),
                                                          context.currentTime(),
                                                          priceUpdate));
                LOGGER.debug("New price: {}, session {}", priceUpdate, session.getSessionId());

                session.updatePrice(priceUpdate);
            }
        }
    }

```

# Other Demand Response Systems

Important to understand is that The PowerMatcher always optimizes the current state based on the latest information. 

Other Demand Response systems often work based on some central optimization algorithm that executes three steps: Forecasting, Scheduling and Real Time Coordination. In the PowerMatcher core we focus on Real Time Coordination, we believe that Forecasting and Scheduling can be performed locally by each individual agent; which results in much more complex dynamics.

------------------------

Please continue reading on how we can give a cluster an [[Objective|Objective]] and potentially turn it into a Virtual Power Plant...