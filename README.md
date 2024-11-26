# Notes on Integrating Agora with Decentralized Social Media Protocols

## Background on Our Vision

For more details, see:  
[Vision Document](https://docs.google.com/document/d/1VWdhZ6uBQlab9cqlvhtWxywMnsYzXq6q/edit?usp=sharing&ouid=103751232663868193979&rtpof=true&sd=true)

Relevant sections:  
- **State of the Art**  
- **3.3 Innovation**

### Summary of the Problem Space:
- We focus on **zk-anonymous social interaction**, which differs from pseudonymous/public social apps.  
- Preserving true anonymity requires more than a pseudonymous "account."  
- For anonymity, users should store post history and reputation in a zk wallet on their mobile device and send zk-proofs in real time.  
- However, this approach does not support following "people," as it requires pseudonymous interactions. For such cases, an "account" (e.g., represented by a `did:plc` as in Bluesky) is sufficient. This is a different use case for a different audience.  
- Posting directly from a wallet simplifies the UX but requires users to have a wallet and demands new protocols to be built.  

For a detailed thread summarizing this idea, see:  
[Thread on Bluesky](https://bsky.app/profile/nicobao.dev/post/3l3tkmnefvr26)


## How to Integrate Atproto (Bluesky) with RariMe (ZK Identity Wallet)

**Atproto Overview:**  
Atproto relies on a **Personal Data Store (PDS)**, a server that stores cryptographic identifiers and personal data. To connect to your PDS, you need to provide an email address, which is kept off-protocol (not publicly shared).

### Integration Approach:
To integrate RariMe with Atproto, we could:  
1. Fork the [Atproto PDS repository](https://github.com/bluesky-social/pds).  
2. Modify it to support login directly via RariMe, besides email.  

**Proposed Workflow:**
- RariMe dynamically creates new PDS accounts based on shared attributes to protect user privacy.  
- Example: A user wants to post as an anonymous American (aged 20–30, living in NYC) and as a member of a "Controversial Political Organization."  
- This would allow users to seamlessly manage multiple personas without impacting UX.  
- If a nullifier is doxxed, RariMe can detect this and generate a new nullifier linked to a new account behind the scenes.

**Alternative Approach (Simpler):**  
Associate zk proofs with an existing Atproto account. Users would log in with email and then verify their identity, binding the proof to their `did:plc`. This is similar to the current setup in Agora.


## Nostr Support

### Key Differences:  
- **Identifier:** Use ZK proof nullifiers directly as identifiers rather than relying on Nostr keys.  
- **Purpose:** Use Nostr infrastructure solely for broadcasting semantic data (proof + payload describing the specific data a nullifier sends).  

### Advantages:
- Simpler implementation compared to Atproto.  
- More logical for purely zk-anonymous applications.  

### Implementation Details:
- Use IPLD, Lexicon, or JSON-LD to define semantics. (Bluesky also requires a Lexicon.)  
- Algorithmically, Nostr is less advanced than Bluesky but does allow custom feed creation.


## Supporting Both Protocols (and Others Like Farcaster)

### Vision:
**Rarimo protocol nullifier becomes the core user identifier**, binding to multiple `did:key`s (basic public keys) generated on the client side of the Agora app (using the standard WebCrypto API or secure Android/iOS storage). More or less one did:key per user device (smartphone, browser, etc). These `did:key`s would then be cryptographically bound to:  
- **Atproto `did:plc`** (hosted on behalf of the user).  
- **Nostr Wallet** (if the user opts to post directly from it).

### Workflow:
1. Every piece of data is signed with the `did:key` and sent to Atproto and Nostr infrastructure.  
2. This data is cryptographically verifiable as originating from a single nullifier.  
3. Modify Nostr to pull data from RariMe nullifiers rather than directly from Nostr wallet keys.  

Alternatively, without changes to Nostr, it could pull data from an "Agora-owned Nostr public key."

### Algorithms:
- **Feed Customization:** Create feeds based on specific attributes (e.g., Ukrainians). Data origin and protocols are *irrelevant* since verifiability lies in the data itself. **This is quite powerful**.
- **Feed Aggregation:** Aggregate posts from multiple protocols, including centralized platforms, ensuring data integrity via the cryptographic chain of signatures ultimately bound to the RariMe ZK proofs.

### Future Enhancements:
- Add **ZK reputation systems** to enhance trustworthiness.
