*For those interested in general information on the Why and How of PowerMatcher please dive into [the PowerMatcherSuite website](http://flexiblepower.github.io/).*

The PowerMatcher is a smart grid coordination mechanism. It balances distributed energy resources (DER) and (flexible) loads. It can also be herded among Demand Response systems, but with a radical new vision. A vision that we call Smart Transactive Energy.

#The PowerMatcher concept

The PowerMatcher concept is based on the micro-economic principle of demand and supply. 
Supply and demand is one of the most fundamental concepts of economics and it is the backbone of a market economy. 
 
The relationship between demand and supply is the underlying force behind the allocation of resources. In market economy theory, demand and supply theory will allocate resources in the most efficient way possible.  
When supply and demand are equal (i.e. when the supply function and demand function intersect) the economy is said to be at equilibrium.

The same mechanism is used in the PowerMatcher.  **The PowerMatcher core application provides the market mechanism for the determination of the market equilibrium, while the devices work as actors for demand and/or supply.** Each device, whether this is a washing maschine, windmill, or industrial turbine can express his willingness to consume or produce energy in the form of a bid (a demand or supply _relationship_). All bids come together in the market mechanism of the PowerMatcher and result in an equilibrium price.

# A small thought exercise

Try to picture a normal auction where a potential buyer would bid on an object e.g. someone bidding on a Van Gogh painting. This process would iterate a couple of times because the buyer does not want to 'show all his cards' in order to get the cheapest price possible. 

A PowerMatcher "auction" does _NOT!!_ work that way... 

In the PowerMatcher protocol **a Bid** describes the device's **entire demand or supply relationship**. The device agent lays out all his cards on the table on _every bidding occasion_ and updates his demand or supply relationship based on the state of machine (e.g. low battery wants to charge). As a result the Auctioneer has all the demand and supply information in the market and requires zero iterations to determine the market equilibrium.  

# EF-Pi integration 

If you want to make your life easier; after studying the concepts of the PowerMatcher it is advised to look into EF-Pi. EF-Pi will greatly reduce you effort in connecting devices.
