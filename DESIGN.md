# Content-Addressable Storage (CAS)

_draft / unfinished_

*Content-addressable storage* --[Wikipedia: Content-addressable storage][]

[Opinions][] and [Document][] both make heavy use of references to document content by hash value. [Messaging][] might rely entirely on the behavior of a distributed CAS to publish its content.

We should identify the minimum interface we require for data storage and retrieval, then examine what existing software might be employed to implement it.

The most basic requests might be to store data locally for future retrieval:

    put(data) -> hash
    get(hash) -> data
    exists(hash) -> bool

Both of these functions may have additonal implications which require more arguments, return values, or environment settings. Let's explore them.

## Data storage

A `put` request might offer data first to local cache, then store in configured local storage areas according to some policy, and then offer the data to configured network-based storage systems.

- Data to be stored might be critical to store for a long time, to backup, etc., while other data might be expirable. How do we manage this? (Management is perhaps out of scope, to be dealt with by each subsystem, provided an expiry or storage intent.)
- The user might want to mark some data "private." Such data should never be made accessible to others. This may mean never storing it in the network (without sufficient encryption), and never serving it to (unauthenticated) users on the network.
- TODO: Messaging may wish to ensure data is served to the network upon request, or deleted from a local node after first retrieval

## Data retrieval

A `get` request might look first in a local cache, then in all configured local storage areas, then it might query all configured network-based storage systems. Considerations:

- The client might have a set of URLs that may be employed to retrieve the data if it is not found locally.
    - The likelihood that data will be available from given URLs is higher than that of it being available on some general-purpose network CAS, so maybe prefer these methods of retrieval?
    - Allowing others to dictate which resources will be queried is an attack vector. There should be a carefully constructed policy about types of URLs, allowed domains and schemas, queries per unit time, etc.
- The user might not want to query the network, if doing so inappropriately reveals information about themselves to others.
    - This should be an option set per-call, default to not querying
- The user might want to query only a trusted subset of the network, like their Avatar's friends in a darknet. They might want to allow their request to propagate throughout the darknet using transitive trust, or to be limited to their Avatar's immediate friends only.
- The user might want to query only networks which are accessed through an anonymizing proxy like [Tor][].
    - This is probably equivalent to "not querying if doing so reveals information about themselves to others" when the user has no network CAS providers configured who declare anonymous=True.
- The user might not want to retrieve data above a certain size if they are using a metered or slow connection.
    - This implies that we might want to limit some queries to network CAS systems that can provide content-length metadata.
- "Querying the network" is distinct from "using Internet bandwidth." The user might maintain a private data store off-site.
- Any values returned should be cached.

Interfaces to network CAS systems should expose queriable properties representing:

- bandwidth: "local" vs. "local or network"
- query propagation: "private" vs. "friends only" vs. "public"
    - this could be boolean "trusted" vs. "public"
- secrecy: "identifiable" vs. "anonymous"

## Result

So far it seems that we need to provide an API like:

    get(hash, local_only=True, trusted_only=False, anonymous=False)
    put(data, private=False, expires=None)

## Storage systems we might employ

- Any number of distributed hash tables; see [Wikipedia: Distributed hash table][]
- Any number of distributed key-value stores; see ["Anti-RDBMS: A list of distributed key-value stores"][Jones 09]
- Postgresql as a key-value store.
- A home-grown filesystem-based mechanism like git uses.
- [IPFS][], "The Permanent Web," "a global, versioned, peer-to-peer filesystem."

## Notes

### Survey of existing CAS systems

[Venti][]
: Data broken into blocks 512 < s < 56k bytes before 

### Composite substrings as responses

Within a string of bytes, every possible substring of bytes corresponds to its own unique hash value. Rather than store and return a large string of bytes, might a CAS for efficiency return a special response which represents the hash values corresponding to multiple other strings, which it might not necessarily store itself, which the client would be responsible for retrieving and concatenating? 

While the probability of common substrings among possible strings increases as the length of substrings decreases, the mechanism sees diminishing returns when the length of the hash value is longer than the substring under consideration. (In some cases it might still be worthwhile; if the sum of the length of all the hash values is less than the sum of all the lengths of the substrings.)

To discover if a string being stored in a CAS should be decomposed into its constituent substrings is equivalent to solving the [longest common substring problem][].

This may be an entirely bad idea:

- Malevolent CAS could perform a DOS against a client by replying with "composite" responses forever; there's no mechanism to prove that the concatenation of the data returned by the hashes will result in the target document.

- If the goal is simply to reduce duplication on the CAS, there are already mature systems to do so. Or, the denormalization could occur on the CAS and the full string always provided to the client.

[longest common substring problem]: http://en.wikipedia.org/wiki/Longest_common_substring_problem
[TOR]: https://www.torproject.org/
[Opinions]: /datagrok/opinions
[Document]: /datagrok/dialogue/blob/master/spec/document-v01.md
[Messaging]: /datagrok/dialogue/blob/master/messaging.md
[IPFS]: http://ipfs.io
[Jones 09]: http://www.metabrew.com/article/anti-rdbms-a-list-of-distributed-key-value-stores
[Wikipedia: Distributed hash table]: http://en.wikipedia.org/wiki/Distributed_hash_table
[Wikipedia: Content-addressable storage]: http://en.wikipedia.org/w/index.php?title=Content-addressable_storage
[Venti]: http://en.wikipedia.org/wiki/Venti
[Named Data Networking]: http://named-data.net/publications/van-ccn-bremen-description/
