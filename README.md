# Datagrok Storage

_draft/unfinished/vaporware_

*Storage* is an abstraction, interface definition, and reference implementation for several varieties of content-addressable storage systems. It describes a system for local caching, archival, serving, distribution, and discovery of pieces of data that are named and referenced by their cryptographic hash digest (hereafter _hash_).

## The big idea

Any finite string of data (hereafter _document_) has a corresponding fixed-length _hash_. With good enough hash functions, hashes are _effectively unique_ for the data they represent.

If we refer to the document we wish to obtain by its hash, figuring out how and from where to obtain it can be the system's responsibility.

Any system that stores content based on its hash is called _content-addressable storage_, or sometimes _associative storage_ or _fixed-content storage_. Traditionally, these terms are used exclusively for systems that store data on local disks. But, we can also refer to documents by their hash when storing them in a cache, in the cloud, or in a peer-to-peer distribution network.

When clients don't specify where the data comes from, we obtain several very cool advantages over referring to them by a location and invented name:

- It doesn't matter if the data comes from a private archive, a trusted third party, or the seedy depths of the Internet: we can be certain the document has not been tampered with because we can check that it produces hash that we asked for. 

- If two clients store the same document, it only needs to be stored once, saving space and bandwidth.

- If two clients request the same document, it only needs to be retrieved once, after which it may be cached.

- Structured documents such as e-mail messages may be broken into their constituent sub-documents for storage. This means you won't have thousands of copies of the company logo from someone's e-mail footer eating up space in your mail archive.

- If a document is modified, its name changes. Those who have a reference to the old document may avoid losing access to it. Document creators can't perform a bait-and-switch undetected.

- Reasoning about how to cache documents becomes easy; most caches may safely exist "forever."

- Hashes are hard to guess or enumerate. _In some situations,_ it may be completely safe for multiple users to share a storage system without complex security measures.

This is just to name a few; this is by no means a complete list.

## Policies

The examples above demonstrate that content-addressable storage has some great advantages. But there are some places were we must be careful:

- Some documents are strictly private; they must never be shared with the network.

- Simply requesting a hash reveals some information about us. Some requests must never be made to the network.

- Some documents are ephemeral and may be forgotten after download; there's no need to store them in a long-term archival system.

- In a peer-to-peer distribution network, storage is not guaranteed, but for some of our documents we want to ensure that others can always find and retrieve them.

- While in some peer-to-peer distribution systems it is impossible to "un-publish," we will want to be able to tell our system that we wish to stop serving any particular document at any time.

- Given the advantages and space-savings of content-addressable systems, some systems enact a policy that data, once stored, is never deleted. This might not always be desired; sometimes we wish to use a storage system that implements a garbage-collection mechanism.

Identifying and implementing solutions for such questions is one of the primary goals of the Storage project.

## Users first

My projects [Opinions][], [Dialogue][], and [Storage][] are built with the intention of being free and open specifications **focused solely on end-user utility**. It embraces and tries to facilitate various parts to be swapped out for competing implementations, at the user's pleasure.

## Similar projects

- [Named Data Networking][] or "Content-Centric Networking," an ambitious, NSF-grant-funded project that is part of the NSF Future Internet Architecture Program. _TODO: How is it different from Datagrok Storage?_

## License

- All documentation herein is [GNU Free Documentation License][] v1.3 unless otherwise noted.
- All source code herein is [GNU Affero General Public License][] v3 unless otherwise noted.

[Named Data Networking]: http://named-data.net/

[GNU Free Documentation License]: http://www.gnu.org/licenses/fdl.html
[GNU Affero General Public License]: http://www.gnu.org/licenses/agpl.html

[Opinions]: https://github.com/datagrok/opinions
[Dialogue]: https://github.com/datagrok/dialogue
[Storage]: https://github.com/datagrok/storage
