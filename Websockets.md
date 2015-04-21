The PowerMatcher is distributed in nature.

Because of the distributed nature of the PowerMatcher the concept is extremely scalable. You can implement it on household level and have an auctioneer optimize your house. It can also be implemented by an energy collective or aggregator to optimize over a large system.

-----------------------------------------------------------

# Technical Implementation

[[Package:net.powermatcher.extensions.websockets|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.extensions/src/net/powermatcher/extensions/connectivity/websockets]]

This package contains the main packages which enable the Powermatcher to communicate over websockets. Websockets is a flexible open communication mechanism which utilizes HTTP as a communication mechanism. The goal of this functionality to allow agents running in environments which are separated from the Auctioneer. This allows for a more scalable system (ie. more servers) or agents running in field stations or residential areas.

The following diagram provides a overview of a possible use case.

![websockets.jpg](websockets.jpg)
**Figure 1: Remote communication using websockets**

Remote communication is hidden for normal agents like a Concentrator, because the MatcherEndpointProxy and AgentEndpointProxy behave like a normal Agent. They have an AgentId and a DesiredParentId (in case of a MatcherEndpoint). The proxy agents add additional logic to handle the remote communication.

An AgentProxyEndpoint has to supply the desired matcherEndpointProxy as a configuration parameter. This value will be communicated to the websocket endpoint via a queryParameter. The websocket endpoint validates the existence of the MatcherEndpointProxy prior to accepting a connection.

The websockets implementation is provided by Jetty, which contains an implementation conform the latest (final) websockets specfication. Jetty 9 is required for websockets and this poses an issue with the default Apache Felix Jetty which is version 8. Currently it's not possible to upgrade the Felix Jetty bundle, so Powermatcher Extensions ships with separate Jetty 9 bundles including required websocket bundles. The websockets.bndrun descriptor contains all required bundles to install and run.
This causes two Jetty servers to run within one OSGI runtime (Felix Jetty and standalone Jetty), the Felix Jetty for the Console has been moved to port 8181. The websocket communication will run over port 8080.

**PowermatcherWebSocketServlet:**
The Servlet which hosts the websocket endpoint. This endpoint automatically registers itself in Jetty to activate the websocket functionality.

**PowermatcherWebSocket:**
Websocket endpoint implementation which receives new connections from external AgentEndpointProxies. When a new AgentEndpointProxy tries to connect, the local MatcherEndpointProxy is associated within the OSGI runtime. The MatcherEndpointProxy receives the websocket session object which it can use to communicate with the external AgentEndpointProxy.

Incoming messages are handed off to the desired local MatcherEndpointProxy. A message contains JSON payload and incomming messages will contain a BID. The encoding / decoding of the messages is handled by the net.powermatcher.extensions.websockets.data and net.powermatcher.extensions.websockets.json components.

Agents which disconnect will also be handled by the PowermatcherWebSocket component. The websocket session will be disconnected and the local MatcherEndpointProxy is notified of this fact.

**AgentEndpointProxyWebsocket:**
Extension of the Core BaseAgentEndpointProxy which implements the specific websocket functionality.

**MatcherEndpointProxyWebsocket:**
Extension of the Core BaseMatcherEndpointProxy which implements the specific websocket functionality.

### Package:net.powermatcher.extensions.websockets.data
This package contains the data classes which represent the DTO objects used in websocket communication. The objects match the usual Powermatcher API objects with one exception: ClusterInfoModel.

**ClusterInfoModel**
Since the CusterId and MarketBasis are only known when the local MatcherEndpointProxy is connected to it's desired parent, this information is communicated asynchronously by the BaseMatcherEndpointProxy.

### Package:net.powermatcher.extensions.websockets.json
This packages contains helper classes to encode / decode the DTO objects to JSON. The Google GSON bundle is used as JSON implementation.

--------------------------

### [[Package: net.powermacther.api.connectivity|https://github.com/flexiblepower/powermatcher/tree/master/net.powermatcher.api/src/net/powermatcher/api/connectivity]]

**AgentEndpointProxy:** Defines the interface and contains all basic methods needed to enable remote communication between an AgentEndpoint and a remote MatcherEndpoint through a MatcherEndpointProxy. The means of communication is up to the implementing class.

**MatcherEndpointProxy:** Defines the interface and contains all basic methods needed to enable remote communication between a MatcherEndpointProxy and a remote AgentEndpoint through an AgentEndpointProxy. The means of communication is up to the implementing class.