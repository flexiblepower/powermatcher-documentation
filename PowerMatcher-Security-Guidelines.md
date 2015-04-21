# Introduction
Smart energy systems – like most complex information systems – deal with sensitive data and therefore require security and privacy preservation measures. In this section, the privacy and security guidelines for PowerMatcher are described for the most important risks that were identified thus far. 

## Identified risks and required measures:   
**1. Device agents on end user hardware cannot be trusted**  
We should validate bids and detect malicious agents  
**2. High impact of integrity of information assets**  
Identification of agents and secure communication is important   
**3. High impact of availability of information assets**  
Central components should be robust
Decentralized components should have alternative desired behaviour when connection with central components is lost  
**4. Criminals can benefit financially from manipulating the cluster**  
We should take this in consideration when choosing a billing solution

### Device agents on end user hardware cannot be trusted
We should validate bids and detect malicious agents.Integrity should be guaranteed by detecting malicious agents and bids.
Hereto: 
* Monitoring of agent, bid and price behaviour is available through the KLE stack (LINK)
* A fraud detection observer will be available (LINK) 

### High impact of integrity of information assets
Identification of agents and secure communication is important. The following measures are taken: 

* Data communication is secured by using SSL
* Agent identification is secured in a three step approach: Definition of proxies; Allowing remote communication; Storage of remote Agent IDs in database (LINK). 

### High impact of availability of information assets
Central components should be robust. 
* Robustness of central components is secured by component design: Components designed based on whiteboard pattern principle. Connection failure of one agents has limited influence on overall cluster. 
* Unit tests are available for central components: agent, concentrator, auctioneer, session manager (link).   

Code quality management is enforced by: 
* Development in development branch
* Peer2Peer review through pull requests, using e.g.unit and integration tests. 
* Merge only possible after pull requests are approved. 
* SonarCube is used to monitor code quality (technical debts, complexity, possible bugs)

Decentralized components should have alternative desired behaviour when connection with central components is lost: 
* Alternative behaviour for most household agents (e.g. fridges, heat pumps) is taken over by the normal controllers of these household appliances. For charge stations, continuation of charging at a minimum charge speed is recommended. 

### Criminals can benefit financially from manipulating the cluster
Users should take this in consideration when choosing a billing solution. Integrity should be guaranteed by detecting malicious prices.Hereto: 
* Monitoring of price behaviour is available through the KLE stack. 
* A fraud detection observer will be available (LINK) 

In general, data management should be secure. Hereto:  
* Data should be stored in a secure way
* Data should be processed in a secury way
* It should be clear for what purpose data is collected
* It should be clear how long should be retained and for what reasons
* It should be clear who owns the data

(Financial) manipulation of the cluster is further prevented by: 
* Limiting information and data access and disclosure to authorized resources and preventing access by or disclosure to unauthorized resources

