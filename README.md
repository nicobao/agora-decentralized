# Notes on Integrating Agora with Decentralized Social Media Protocols

## What is Agora?

See https://agoracitizen.network

## Background on Our Vision

For more details, see:  
[Vision Document](https://docs.google.com/document/d/1VWdhZ6uBQlab9cqlvhtWxywMnsYzXq6q/edit?usp=sharing&ouid=103751232663868193979&rtpof=true&sd=true)

(our vision evolved since then though!)

Relevant sections:  
- **State of the Art**  
- **3.3 Innovation**

Currently we use RariMe as follows:
- Device self generates a keypair via standard webcrypto API or android/iOS secure storage and derives from it a `did:key`
- Agora requests a proof
- User sends proof with RariMe app, proof contains a unique nullifier
- Agora associates this nullifier with the `did:key`
- login from another device means generating a new did:key and associating a new RariMe proof to it with the same nullifier 
- users then sign all their content with the did:key seamlessly and securely from Agora frontend without needing any external wallet dependency nor specific user interaction
- we broadcast these signed data and ZK proofs to Nostr. Currently we don't broadcast actual data, just hashes, so users can easily request data deletion from trusting our server.

### Summary of the Problem Space:
- We focus on **zk-anonymous social interaction**, which differs from pseudonymous/public social apps.  
- Preserving true anonymity requires more than a pseudonymous "account."  
- For anonymity, users should store post history and reputation in a zk wallet on their mobile device and send zk-proofs in real time.  
- However, this approach does not support following "people," as it requires pseudonymous interactions. For such cases, an "account" (e.g., represented by a `did:plc` as in Bluesky) is sufficient. This is a different use case for a different audience.  
- Posting directly from a wallet simplifies the UX but requires users to have a wallet and demands new protocols to be built.  

For a detailed thread summarizing this idea, see:  
[Thread on Bluesky](https://bsky.app/profile/nicobao.dev/post/3l3tkmnefvr26) or refer to the [Vision Document](https://docs.google.com/document/d/1VWdhZ6uBQlab9cqlvhtWxywMnsYzXq6q/edit?usp=sharing&ouid=103751232663868193979&rtpof=true&sd=true).

## Integrate Atproto (Bluesky) with RariMe (ZK Identity Wallet)

**Atproto Overview:**  
Atproto relies on a **Personal Data Store (PDS)**, a server that stores cryptographic identifiers and personal data. To connect to your PDS, you need to provide an email address, which is kept off-protocol (not publicly shared).

To integrate RariMe with Atproto, we have multiple options. 

### Option 1 - RariMe first:
- login with RariMe on Agora like we do right now on Agora , but we also create a corresponding PDS and host it for you. We can bind our `did:key`s to the `did:plc` with simple signatures.
- optionally users can instead associate their existing atproto account
- this requires:
    - Fork the [Atproto PDS repository](https://github.com/bluesky-social/pds).  
    - Modify it to support login directly via RariMe, besides email

### Option 2 - same as option 1 but with enhanced privacy protection:
- multiple RariMe nullifiers from the same RariMe account bound to different ID attributes = creation of multiple Agora accounts created on the fly = creation of multiple PDS accounts
- Agora frontend dynamically request the creation of new Agora & associated PDS accounts based on shared attributes to protect user privacy.  
- Example: A user wants to post as an anonymous American (aged 20–30, living in NYC) and as a member of a "Controversial Political Organization."  
- This would allow users to seamlessly manage multiple personas without impacting UX.  
- If a nullifier is doxxed, either Agora or RariMe can detect this and request the generation or generate a new nullifier linked to a new account behind the scenes.

### Option 3: simplest approach, atproto first: 
- Associate zk proofs with an existing Atproto account. Users would log in with atproto via email and by entering their handle (classic, that's their primary identifier), and then verify their identity, binding the proof to their `did:plc`.
- does not require forking atproto PDS at all


## Integrate Nostr with RariMe (ZK Identity Wallet)

### Classic integration:
- associate ZK proof nullifiers to Nostr Wallet key, broadcast that
- then post from Nostr normally 

### Approach to Nostr solely as a neutral infrastructure (what we do now): 
- Use ZK proof nullifiers and our `did:key`s directly as identifiers rather than relying on Nostr keys.
- Use Nostr infrastructure solely for broadcasting semantic data (user's signed data and proof + payload describing the specific data a nullifier sends). We signed the Nostr data with a Nostr pub key that is owned by Agora backend.

### Hybrid approach:
- associate Nostr Wallet pub key to the user's `did:key`s that is bound to the ZK proof and its nullifier, broadcast that to Nostr
- (cross) post normally to Nostr from Nostr Wallet

## Supporting both protocols and others like Farcaster and ActivityPub (Mastodon) all at once: cross-posting, cross-identities, feed algorithm choice combining data from multiple social media protocol

### Vision:
**Rarimo protocol nullifier becomes the core user identifier**, binding to multiple `did:key`s (basic public keys) generated on the client side of the Agora app (using the standard WebCrypto API or secure Android/iOS storage). More or less one did:key per user device (smartphone, browser, etc). These `did:key`s would then be cryptographically bound to:  
- Atproto `did:plc` PDS (registered and hosted on Agora backend on behalf of the user OR associated by the user manually if they already had one).  
- Nostr Wallet (if the user opts to post directly from it).

did:keys are useful because they use built-in device crypto algorithms, that does NOT require any user interaction to create signatures, while staying secure. They enable not to force users to constantly create proofs from RariMe.

### Posting:
1. Every piece of data is signed with the `did:key` and sent to Atproto and Nostr infrastructure (cross-posting)  
2. This data is cryptographically verifiable as originating from a single nullifier.  

### Algorithms:
- **Feed Customization:** Create feeds based on specific attributes (e.g., Ukrainians). Data origin and protocols are *irrelevant* since verifiability lies in the data itself. **This is very powerful: Humans as the API instead of some protocol**.
- **Feed Aggregation:** Aggregate posts from multiple protocols, including centralized platforms, ensuring data integrity via the cryptographic chain of signatures ultimately bound to the RariMe ZK proofs.

### Requirements:
- Use tech such as IPLD, Lexicon, or JSON-LD to define semantics, and make this interoperable between social protocols.

### Future Enhancements:
- Add **ZK reputation systems** to enhance trustworthiness.
- Making RariMe compatible with ZKEmail so as to allow recovery of the RariMe private key with familiar UX.

## Current conclusion for Agora (in progress)

- We keep our login with RariMe and associate it with the `did:key mechanism`
- Users can choose to associate their atproto account to these did:key and ZK proof, otherwise Agora would default to registering and storing a new PDS on their behalf. Email is made optional (requires forking PDS).
- Users can add their own Nostr Wallets and associate these `did:key` and RariMe proofs to it
- when users post, we crosspost data to atproto and Nostr, if they added a Notr wallet 
- data is defined extensively using semantics, especially the compatibility between protocols
- we use these semantics to create cross-protocol feed aggregators (same as bsky for algorithmic choice but instead of limiting the data origins to being on atproto we can integrate other protocols)