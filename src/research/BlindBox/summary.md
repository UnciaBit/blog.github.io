# BlindBox: Deep Packet Inspection over Encrypted Traffic
-----
**Background**
BlindBox [24], first enabled an outsourced middlebox for DPIs without decrypting the traffic in the middle by using the technology of garbled circuit (GC) 

**Goal of BlindBox**
Demonstrate the possibility of coexistence of middleboxes and encryption

-----
problem statement, motivation, background, related work, the detailed techniques and protocols, security analysis, evaluation results, pros and cons.

## Introduction

**Uses of Deep Packet Inspection (DPI) in middleboxes**
- Network Intrusion Detection/Prevention (IDS/IPS)
	- Detects packets from compromised sender
- Exfiltration prevention devices
	- Block accidental leakage of private data
	- Searching for document confidentiality watermarks in the data transferred out of enterprise network
- Parental filtering devices

**There exists:**
- HTTPS and other encryption protocols
- Which prevents eavesdroppers from viewing user's private data through E2E encryption
- But this includes middleboxes

**The problem of E2E encryption like HTTPS:**
- Middleboxes cannot inspect payloads thus cannot perform their task as a middlebox
- If you attempt to enable middlebox processing (MITM attack on SSL to decrypt traffic at the middlebox)
	- You violate the E2E security guarantees of SSL
	- Concerns that private data may be logged at the middlebox

**Dilemma**: Functionality of middleboxes or privacy/security from encryption

BlindBox attempts to solve this problem by performing the inspection directly on the encrypted payload, without decrypting the payload at the middlebox.

**Challenges in creating this system:**
- Cryptographic operations has to be quick enough it doesn't hinder the network operation
	- Especially when middleboxes require support for 'rich operations' like matching regular expression
- Existing cryptographic systems like below are slow, which decreases the network rates
	- Fully homomorphic
	- Functional encryption

**Features of BlindBox**
- Blindbox specialises on the network setting
- Offers two classes of DPI computation each with its own privacy guarantees
	- Exact match privacy
	- Probable cause privacy
- Both of these privacy models are stronger than the MITM approach that is currently being used
- Provides similar security guarantee as existing searchable encryption through use of randomised encryption schemes
- Depending on the class of computation, middlebox can learn a small amount of information about the traffic
	- To detect rules efficiently

**Classes of computation:**
- 1: DPI application rely only on exact string matching (Watermarking, parental filtering, limited IDS
	- Exact match privacy model used:
		- Middlebox only learns the positions of the attack keywords in the flow
		- If the substring of the flow does not match the attack keyword, the middlebox learns virtually nothing
- 2: Supports all DPI applications (regular expressions, scripting etc...)
	- Probable cause model used:
		- Middlebox can see an individual packet (decrypted)
		- Or the flow if the flow is suspicious
			- Condition: Flow contains a string that matches the known attack keyword
		- If the stream is not suspicious, the middlebox cannot see the decrypted stream
- Use can choose one of these privacy models

**Techniques developed to implement the models:**
- DPIEnc + BlindBox Detect
	- Searchable Encryption Scheme
	- Fast detection protocol
		- Inspect encrypted traffic for certain keywords efficiently
	- Has speed of deterministic encryption
	- And security of randomised encryption
- Obfuscated Rule Encryption
	- Middlebox can obtain encrypted rules based on
		- The rules from the middlebox
		- Private key of the endpoints
	- Without
		- Endpoint learning the rules
		- Middlebox learning the private key
	- Based on garbled circuits and oblivious transfer
- Probable Cause Decryption
	- Mechanism to allow flow decryption when suspicious keyword is found in the flow
	- Used in the probable cause model


Optimal setting for BlindBox: Long or persistent connection through SPDY-like protocols or tunnelling
Non-optimal setting for BlindBox: Short, independent flows with many rules

**Further research:**
- Support other middlebox functions
	- Processing packet headers
	- Modify packet payloads (transcoders/WAN optimisers)
	- Terminate sessions (Web or SIP proxies)

----

## Overview

### MiddleBox Architecture

Four Parties:
- S: Sender
- R: Receiver
- MB: Middlebox
- RG: Rule generator

1. RG generates attack rules (aka. signatures)
	- Each rule describes an attack with fields such as:
		- Keywords to be match in the traffic
		- Offset information for each keyword
		- Regular expression
	- Emerging Threats, McAfee, Sysmantec performs RG roles today
2. S and R send traffic through MB
3. MB allows S and R to communicate unless MB observes attack rule in their traffic

- Middleboxes are deployed as central point of control
	- To enforce security policies in their network (with no control of the endpoints)
	- It is easier to manage and upgrade
- Some research considered edge deployments
	- Where endpoints perform all middlebox processing on their own traffic
	- And rules are distributed to the endpoints
- BlindBox attempts to make more compatible with existing architecture
	- Edge-based model does not keep rules hidden from endpoints
- Currently, MB can read any traffic sent between S and R
- In BlindBox, MB should be able to detect if against rules by RG match the traffic between R and S
	- But should not learn the contents of the traffic that does not match RG's attack rules

### Security and Threat Model

#### System Requirements
- BlindBox must maintain MB's functionality
- Endpoints must not gain access to the IDS rules
	- Prevent IDS evasion
	- Most vendors rely on the secrecy of their rulesets in their business model
		- As they may have more comprehensive, efficient or harder to evade rules
- Middlebox cannot read the user's traffic, except when traffic is flagged suspicious based on the attack rules

#### Threat Model

Two attackers:
- **Original attacker considered by IDS**
	- Same as what the conventional IDS detects as attacker
	- Goal is to detect this attack over encrypted traffic
	- One endpoint can behave maliciously, at least one endpoint must be honest
		- Otherwise
			- If two malicious endpoints agree on a secret key to encrypt the traffic
			- Prevention is impossible by the security properties of the encryption scheme
		- It is the default assumption for exfiltration detection and parental filtering system
- **The attacker at the middlebox**
	- This attacker attempts to extract private data from the encrypted traffic passing through the MB
		- MB performs the detection honestly
		- But it tries to learn private data from the traffic and violate the privacy of the endpoints
		- Assume attacker at MB reads all the data accessible to the middlebox
	- BlindBox needs to hide the content of the traffic from MB
		- While allowing MB to perform DPI
		- Attack rules are not hidden from the MB itself
			- But is hardcoded in the MB
	- 

#### Privacy Models

- **Exact Match Privacy**
	- Guarantees that
		- Middlebox will only discover the substrings of the traffic that are exact matches of known attack keywords. It will only learn the offset in the flow the attack word appears.
		- If traffic did not contain the known attack keywords, it remains unreadable to the middlebox
- **Probable Cause Privacy**
	- Guarantees that
		- Middlebox will be able to decrypt a flow if a substring of the flow is an exact match for a known attack keyword
	- Useful for IDS tasks which require regular expressions or scripting
	- Provides security guarantees similar to searchable encryption (while being faster)

Both models provides stronger security compared to the MITM approach in deployment today, where all traffic is decrypted regardless of suspicion

### BlindBox System Architecture


**Pre-connection: **
1. RG generates a set of rules
	1. Contains a list of suspicious words
2. RG signs the rules with its private key
3. RG shares the rules with MB
4. S and R install BlindBox HTTPS configuration with RG's public key

**Connection Setup: **
1. SSL handshake between S and R agrees on key k_0
2. S and R uses k_0 to derive 3 keys:
	1. k_SSL - SSL key used to encrypt the traffic
	2. k - used in the detection protocol
	3. k_rand - seed for random
3. MB obtains each rule from RG  deterministically encrypted with key k
	1. It occurs in a way such that MB does not learn the value of k
	2. and R and S do not learn the rules
	3. This exchange is called #obfuscated-rule-encryption

**Sending Traffic**
1. Encrypts the traffic with SSL
2. Tokenise the traffic by splitting it in substrings taken from various offsets
3. Encrypts the resulting tokens using the DPIEnc encryption scheme

**Receriving Traffic:**
- At the receiver
	- Decrypts and authenticates the traffic using SSL
	- Checks that the encrypted tokens are encrypted properly by the sender
		- From an assumption that one endpoint is honest, this should prevent attack when one endpoint is malicious

### Protocols

Protocol I and II provide *Exact Match Privacy*, protocol III provides *Probable Cause Privacy*

**Protocol I**
- One keyword
- Detected if this keyword appears at any offset in the traffic, based on equality match
- Suffices for document watermarking and parental filtering
	- But supports only a few IDS rules

**Protocol II**
- Multiple keywords + position information of the keywords
- Supports wider class of IDS rules

**Protocol III**
- Additionally supports regular expressions and scripts
	- Enabling full IDS


----

## Protocol I : Basic Detection

Protocol I matches one suspicious keyword against the encrypted traffic

**Problem**: Searchable encryption, typically used to detect a keyword match on encrypted text, allows the endpoints to see the rules

**Solution**: Use obfuscated rule encryption

</br>

**Problem**: Existing encryption schemes does not meet the security and network performance requirements
- Deterministic schemes
	- Leaks whether two words in the traffic are equal regardless of the rule match
		- Provides weak privacy as it allows frequency analysis
	- Fast because MB builds indexes that can process each token (word) in a packet in time **logarithmic** in the number of rules
- Randomised schemes
	- Prevents frequency analysis through salting ciphertexts
		- Provides stronger security compared to deterministic scheme
	- Slow because combining each token with each rule for salting requires processing time **linear** to the number of rules for each token
		- Impractical for packet processing

**Solution**: Use DPIEnc encryption scheme and BlindBox Detect detection protocol, offers has the speed of deterministic encryption and the security of randomised encryption

**Tokenisation in BlindBox:** 
- Sliding window tokenisation (window-based)
	- For every offset in the bytestream, sender creates a 8 byte token
		- e.g. "alice apple" -> "alice ap", "lice app", "ice appl"
	- MB will be able to detect rule keywords of length 8 bytes or greater
	- For keyword larger than 8 bytes, MB splits it in substrings of 8 bytes
		- e.g. Keyword: "maliciously" | MB searches for: "maliciou", "iciously"
	- Each encrypted token is 5 bytes + endpoint generates one encrypted token per byte of traffic = bandwidth overhead is 5x
	- Optimisations to reduce overhead (For HTTPS only IDS)
		- Ignore tokenisation of images and videos
		- Ignore redundant tokens using delimiter-based matching
			- Match keywords that start and end on delimiter-based offsets
			- e.g. Payload: “login.php?user=alice” | Keywords: "login", "login.php", "?user=", "user=alice" | Not: "logi", "logi.ph"

### DPIEnc Encryption Scheme

- This encryption is used to encrypt each token `t`

- Encryption of DPIEnc is: **`salt, AES_AES_k(t) (salt) mod RS`**

- `r` is the rule/keyword

**In simple deterministic encryption: encryption of `t`  is `AES_k (t)`:**
- To check if `t` equals a keyword `r`
	- MB checks if AES_k (t) == AES_k (r)
- Weak security because every occurrence of t will have the same ciphertext


**H:** 
- To avoid this issue, a 'random function' H and salt is used in combination to produce the ciphertext: `salt, H(salt, AES_k (t))`
- H must be pseudorandom and not invertible
- MB checks for match by computing` H(salt, AES_k (r))` and matching
- SHA-1 is typical for H but is not as fast as AES thus AES is used for BlindBox
	- But AES have different properties
	- So AES must be keyed with a value that MB does not know when there is no match (hence `AES_k (t)`)

**RS:** 
- Reduces the size of the ciphertext to reduce bandwidth overhead
- In BlindBox RS = 2^40 -> ciphertext of 5 bytes

**Detecting matches between keyword r and encrypted token `t`:**
- MB computes `AES_AES_k(r) (salt) mod RS` using salt and knowledge of `AES_k(r)`
- Tests this against `AES_AES_k(t) (salt) mod RS`
- MB does not require the `k` value for the entire process

- MB performs a match test for every token t and rule r
- Performance per token is **linear to the number of rules**
	- Too slow
- Using BlindBox's detection algorithms reduces the cost to **logarithmic of number of rules**


### BlindBox Detect Protocol

Denote: **`Enc_k(salt, t) = AES_AES_k(t) (salt)`**

**Flawed approach:**
- Pre-compute `Enc_k(salt, r)` for every rule `r` and for every possible salt
- Arrange the values in a search tree
- For each encrypted token `t`, MB looks up `Enc_k(salt,t)` in the tree and check for equal value
- It is infeasible to enumerate all possible salts for each keyword
- Using only few salt will lead to weak security because the salt can be reused for the same token
- Therefore every encryption of a token `t` must contain different salt

**Secure approach:**
- The sender generates salts based on the token value and no longer send the salt in the clear along with every encrypted token
- Sender keeps a counter table, mapping each token encrypted so far and how many times it appeard in the stream so far
- Sender sends one initial salt (salt_0) before sending the encrypted tokens -> MB records it
- For each token t, sender sends `Enc_k(salt, t)` but not salt
- When encrypting token `t`, the sender checks the number of times it was encrypted so far in the counter table (`ct_t`)
- Then it encrypts the token with the salt `(salt_0 + ct_t)` by computing `Enc_k(salt_0 + ct_t, t)`
- Hence no two equal tokens will have the same salt
- The sender rests the counter table every `P` bytes sent
	- Then the sender sets salt_0 <- salt_0 + max_t ct_t + 1
	- And announce new salt_0 to MB
- MB creates a table mapping each keyword to a counter `ct*r`, indicating the number of times the keyword appeared in the stream.
- MB creates a search tree containing the encryption of each rule with a salt computed from `ct*r: Enck(salt0 + ct*r, r)`
- When there is a match
	- MB increments `ct*r`, computes and inserts the new encryption `Enc_k(salt_0 + ct*r, r)` into the tree, and deletes the old value

In summary:
- MB holds:
	- Counter `ct*r` for each rule r
	- Search tree made of `Enc_k(salt_0 + ct*r, r)` for each rule r
- For each encrypted token `Enc_k(salt, t)` in a packet
	- If `Enc_k(salt, t)` is in the search tree:
		- Perform action for this match
		- Delete the node in tree corresponding to `r` and insert `Enc_k(salt_0 + ct*r + 1, t)`
		- Set `ct*r <- ct*r + 1`
- All operations are logarithmic in number of rules

### Rule Preparation

