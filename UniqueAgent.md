In a PowerMatcher cluster, it is not allowed to add an agent with an agentId which is already registered in the cluster. The sessionManager holds a list with registered agentId’s.

---------------------------------------

# AgentId Uniqueness

![UniqueAgentId](UniquenessAgents.png)
**Figure 1: Agent Uniqueness**

When an agent wants to connect to the cluster, the sessionManager will check the uniqueness of the agentId.
If there is no agentId connected to the cluster with the same agentId, the connection will be made successfully. 

However, if the agentId aready exists, PowerMatcher will log that the agentId is already registered in the cluster and doesn’t allow the connection.

When activating the agent in the configuration admin of Felix, it is not possible to add that existing agentId.

# Technical Implemenation

In the [[Session Manager|https://github.com/flexiblepower/powermatcher/blob/master/net.powermatcher.runtime/src/net/powermatcher/runtime/sessions/SessionManager.java]]...

When a MatcherEndPoint is added, it checks if it already exists:

```
    public void addMatcherEndpoint(MatcherEndpoint matcherEndpoint) {
        String agentId = matcherEndpoint.getAgentId();

        synchronized (this) {
            // Check for duplicate
            if (matcherEndpoints.containsKey(agentId)) {
                LOGGER.warn("MatcherEndpoint added with agentId " + agentId
                            + ", but it already exists. Ignoring the new one...");
                return;
            }
```

And when an AgentEndPoint is added, it checks whether the AgentId already exists in the subsystem of the desired Parent (MatcherId):

```
            // Check if it already exists
            for (PotentialSession ps : potentialSessions.get(matcherId)) {
                if (agentId.equals(ps.getAgentId())) {
                    LOGGER.warn("AgentEndpoint added with agentId " + agentId
                                + ", but it already exists. Ignoring the new one...");
                    return;
                }
            }
```
