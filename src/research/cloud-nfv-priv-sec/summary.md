----

P. Jiang et al., "Building In-the-Cloud Network Functions: Security and Privacy Challenges," in Proceedings of the IEEE, vol. 109, no. 12, pp. 1888-1919, Dec. 2021, doi: 10.1109/JPROC.2021.3127277.

[Available here](https://ieeexplore.ieee.org/document/9645060)

----

# Building In-the-Cloud Network Functions: Security and Privacy Challenges



including problem statement, motivation, background, related work, the detailed techniques and protocols, security analysis, evaluation results, pros and cons.
## Glossary

Network function virtulisation (NFV) - Replacing network appliance hardware with virtual machines. Load balancing, routing and firewall security are all performed by software instead of hardware components.

Middleboxes - Device that inspects, filters, and manipulates traffic for purposes other than packet forwarding

Stateful packet filtering - Filters/monitors the network to ensure the traffic is legitimate

## Summary

### Abstract

**Issue:**
- NFV through could service provider, redirecting communication traffic to a third party service provider has security and privacy concerns
- End-to-end encryption can protect traffic but hinders data usability

**Agenda of the paper:**
- Survey on network function outsourcing with focus on privacy and security issues

### Introduction: NSVs and their challenges and security risks in cloud

**Pre-NSV:**
- Traditional telecommunication industry depended on specialised hardware equipment of different manufacturing standards
	- This leaded in poor product development cycle

**Functions of NSV:**
- Attempts to solve the complications in deploying networking and communication hardware infrastructures
- By transferring communication data to standard hardware (servers, switches, and storage) in network nodes
- This also simplifies the design and operation of network functions
	- Increased flexibility, programmability, and expandability
- Applications of NSVs include
	- Switching
	- Mobile Network Nodes
	- Tunnelling Gateway Elements
	- Traffic Analysis
	- Virtualised Home Environment
	- Security functions

**Recent Trend:**
- Outsource virtualised network functions to the cloud
- Mitigate the difficulty of provisioning and management of specialised hardware

**Examples of network functions (=middleboxes):**
- Firewalls
- Deep packet inspection (DPI)
- Load balancers (LB)

**Examples of cloud network functions provider:**
- Amazon CloudFront
- Google Cloud Networking
	- NAT
	- Load balancing
	- Routing
- VMWare Ready for Telco Cloud

**Advantages of using cloud infrastructure for network functions:**
- High performance packet packet processing
- No network hardware configuration and management

**Disadvantages of using cloud infrastructure:**
- Network function is outsourced -> Traffic is redirected to the cloud
	- Cloud servers has root access to outsourced functions and communication traffic
	- Vulnerable to internal threats
		- Abuse the root access
- The traffic may contain confidential information
	- Violates the user's privacy
- Outsourced function may contain proprietary assets
	- E.g. inspection rules for intrusion detection systems
	- Commercially sensitive data could be exposed if deployed in insecure environment

**Current secure communication solutions and their issues:**

End-to-end encryption is used to protect data in transit but:
- Hinders the processing and computation of the packets and rules at the middlebox
- But attempting to avoid this issue by decrypting at middlebox violates E2E security

**Ongoing research to enable E2E with middleboxes**
- Research in designing a new TLS protocols that are compatible with middleboxes
- Enables traffic-owner-controlled decryption at middleboxes
	- Partial traffic is still visible to middleboxes
- This method is practical and systematic but lacks in security
- Middleboxes in designs discussed in this surveys are:
	- Never supposed to learn the plaintext content of the traffic
	- But still inspect the encrypted traffic

**Basic process of network functions**

Matching with progenerated rules -> computing -> getting the action to be performed on the packets  

**Goals of privacy-focused network function outsourcing:**
1. Complete general network functions (cheaply on an encrypted traffic)
2. Preserve the privacy of the traffic and/or rules
3. Verifiable computation on middleboxes


This survey looks at different secure in-the-cloud network functions, more details are given on [page 3 on the article](https://ieeexplore.ieee.org/document/9645060)



### NFV and the Outsourcing Practice

**Problems with traditional network architecture**
- Incompatibility of different network hardware devices
- Middleboxes designed and implemented under different standards
	- Hinders network application development and maintenance
- Costly to implement new network functions by adding new middleboxes in an existing network architecture

**How NFV attempts to solve this problem**
- Standardise middleboxes by consolidating various network equipment (servers, switches, storages in network nodes and datacenters)


**NFV architecture consists of:**
- Virtualised network functions
- NFV infrastructure
	- Virtual compute,
	- Virtual Storage
	- Virtual network
	- Other hardware resources
- NFV Management and orchestration

**Different types of network functions:**
- Deep Packet Inspection (DPI)
	- Network traffic analysis technique that performs:
		- Traffic analysis
		- Inspection
		- Filtering
	- Inspection rule for the middleboxes are provided by the rule generator
	- Rules in DPI may involve detecting/inspecting the IP header, as well as the application layer payload
	- DPI middlebox compares the incoming packets with the inspection rule
		- If there is a match, middlebox performs certain actions (accept, drop) according to the rules
	- It can be used for malicious network traffic detection, precise advertising etc...
- Intrusion Detection System (IDS)
	- Prevents malicious access to internal networks
	- Three variations exists:
		- Signature-based detection
			- Uses pattern matching to detect malicious traffic
			- Includes two steps:
				- Preprocessing: Packets are decoded and reassembled into IP packet fragments
				- Attack signature matching: IDS conducts multistring pattern matching to detect specific keywords
			- If nothing is found, flow is tagged as "innocent"
			- Else IDS performs further signature detection
				- Metadata pattern match
				- String pattern match
				- Binary pattern match in the payload
		- Anomaly-based detection
			- Uses machine learning to train a model to differentiate traffic
		- Reputation-based detection
			- Scores each traffic flow to recognise potential threats
- Firewall