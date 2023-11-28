# External references for CBOR

This document specifies a Concise Binary Object Representation tag for representing data that is physically stored externally to the CBOR data item.

Tag: 32,769  
Data item: multiple  
Semantics: External reference  
Point of contact: Christopher Head <chead@chead.ca>  
Description of semantics: https://gitlab.com/Hawk777/cbor-specs/-/blob/main/external-reference.md

# Semantics

The External Reference tag can be applied to a data item to indicate that the data item indicates the location where the actual data can be found, rather than encoding the data itself. As the ability to obtain the data from the external source, as well as the form of references to such data, is somewhat application-specific, this tag SHOULD only be used between cooperating entities with a mutual understanding of the meaning of such references.

# Example use cases

- The sender and recipient share a filesystem. The sender wishes to send a CBOR data item to the recipient which includes the contents of a file. Rather than including the file verbatim, the sender includes an external-reference-tagged string holding the filename. The recipient treats the message as if it contained the contents of the file. Much less data needs to pass through the CBOR encoding process, and in the case of multiple computers sharing a network filesystem, the file data only passes over the network once (from the file server to the recipient), rather than twice (once from the file server to the sender in order to perform CBOR encoding, then again as part of the CBOR message from sender to recipient).
- The sender and recipient share a memory mapping (perhaps they are threads within a process or have mapped a shared file). The sender wishes to send a large string to the recipient. Rather than allocating enough writeable memory to hold the CBOR message including the string, then sending this large message to the recipient, the sender instead includes an external-reference-tagged array holding the memory address (pointer) and length of the string. The recipient refers to its own mapping of the same string rather than receiving a copy of the whole string.

# Security considerations

If a recipient does not validate the sender’s authority over the object referred to by an external reference, a malicious sender could convince the recipient to read data to which the sender should not have access. In the absence of proper handling by the recipient, the recipient would read the data under its own authority rather than the authority of the sender, yet consider the data to have originated at the sender.

If a recipient resolves external references via network requests, a malicious sender could convince the recipient to issue a network request to a server under the sender’s control, potentially exposing information about the recipient (as part of the network request) that the sender would otherwise not know.

If a recipient limits the size of a CBOR data item before decoding it, but does not similarly limit the size of external references before resolving them, a malicious sender could exhaust the recipient’s resources.

If a recipient resolves external references via network requests, a malicious sender could cause the recipient to issue a large number of requests to a victim server, causing denial of service.

For these reasons, external references MUST only be used when the sender is trusted, when the method by which references are resolved is inherently free of such concerns, or when such concerns have been appropriately addressed.
