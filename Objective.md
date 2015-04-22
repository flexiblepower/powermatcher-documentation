# Cluster Objectives

The Objective Agent interfaces with the business logic of an external application on one side; and with the PowerMatcher on the other side. 

For example, if you want the PowerMatcher cluster to serve as a Virtual Power Plant (VPP), NOT to balance itself, but instead have it produce or consume **a surplus** amount of energy. This means you have to push the PowerMatcher market out of balance and thus NOT forward the Equilibrium price, instead *manipulate* the price in the PowerMatcher market. This can be done using the Objective Agent.

This is just one example for using the Objective Agent.

---------------------------------------------------------------

# The Objective Agent 

![](objective_agent.png)

**Figure 1 - Objective agent**

The PowerMatcher core contains a special agent called the ObjectiveAgent.

# Technical Implementation
