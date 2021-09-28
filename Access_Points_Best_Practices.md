# Best Practices for Peppol Access Points


## Introduction

This is a collection of best practices for parties that are running Peppol Access points, aimed at increasing the general reliability of the peppol network.

At this time, it is just an unordered list of considerations and advice. If the list grows, we may need to divide it into topic sections.

This document may refer to other documents for specific topics.

## Best practices

### Corner 2: Retries on send errors

When sending documents from an access point to another accesspoint, the transport can fail.

Transport errors may have many different causes; there could be something wrong with the way corner 2 tries to send a document, or with what it is trying to send. There may also be something wrong on the receiving end. Some of these errors may be temporary in nature, and some of these errors may be permanent, in the sense that trying the same message without any alterations will always result in a failure.

To increase the reliability of the network, failed attempts should be retried. However, in the case of permanent errors, or errors caused by the sender itself, simple retries will result in the same failure. Additionally, if the failure is caused by the receiving end due to a temporary problem, such as heavy load, immediate retries might prolong the issue rather than solve it.

#### General error handlign process

Therefore, the error handling process should have the following steps in place:

1. Determine whether the error is caused by the sending or the receiving accesspoint. Abort and report an error if it is caused by the sender.
2. Determine whether the error is temporary or permanent in nature. Abort and report an error if it is a permanent error.
3. Apply an exponential back-off policy for retrying, i.e. after every failed attempt double the time the system waits before trying again.
4. Set a maximum time after which the error is considered permanent and the attempts to send are aborted.

When the process is aborted, corner 1 must be informed, so that it is aware that the document will not arrive at corner 3 and corner 4.

In some cases, it may not be possible to determine the nature of the problem without checking log files on both ends. In that case, it is necessary to contact another service provider. See below for some best practices regarding the information to include when contacting other service providers about transport errors.

#### Specific temporary and permanent error examples

- ebMS errors: Most of the errors from the list in the [ebMS 3.0 specification](http://docs.oasis-open.org/ebxml-msg/ebms/v3.0/core/cs02/ebms_core-3.0-spec-cs-02.html#6.7.Standard%20ebMS%20Errors|outline) can be considered permanent, in that they signal an issue with the message itself. The exceptions are EBMS:0005 and EBMS:0006, which may signal temporary errors.
- A general HTTP 500 error, without a specific AS4 response: The receiving access point appears to be unable to even process the message and create an AS4 message (whether error or success); it is likely this is a temporary error connecting to the backend.
- Timeouts (no response): These can be considered temporary (service unavailable) and the delivery should be retried

### Corner 3: Handling errors while receiving

When an error occurs while receiving documents as a corner 3, it is important to handle the error correctly as well, not in the least so that the sender can properly act on it.

Again, much depends on the nature of the error itself. For instance, in the case of an invalid document, the result could simply be that the message itself is accepted, but an MLR is sent containing an error. An unparseable SBDH can of course result in an immediate AS4 error, but in many cases the problem may be more subtle.

It is therefore a good practice to be as specific as possible in error messages (without giving away unnecessary information), and to proactively check error logs to that issues can be investigated as soon as possible.

Special care should be taken when the document is accepted on corner 3, but delivery to corner 4 fails. In this case, it may not always be possible to automatically inform corner 2 (and corner 1), that the delivery has failed. It is, however, paramount for the reliability of, and the trust in, the network, that the document does not simply 'disappear'.

In most cases, this requires some form of contact between corner 3 and corner 4 to resolve the delivery issue, and corner 4 may need to inform corner 1 that the delivery has failed. A best practice for corner 3 would be to keep the message until the issue is solved, so that corner 1 does not need to send it again. For instance by storing the message in a dedicated error folder.

#### Specific error scenario's

(this is not an exhaustive list)

- Recipient ID not known to receiver: This scenario must be responded to with an EBMS:0004 error, with the error details NOT_SERVICED, as per [specification](https://docs.peppol.eu/edelivery/as4/specification/#_feedback_when_receiver_is_not_serviced).


### SBDH values should match document contents

When using a UBL document type that contains an EndpointID for a sender or recipient, their values should match those used in the SBDH Envelope Sender and Receiver fields.

For instance, you should NOT have a KvK number as the Receiver value in the envelope when the EndpointID value in the AccoutntingCustomerParty of a UBL invoice is an OIN number. 

### Caching SMP results

When sending multiple documents to the same recipient, it makes sense to cache the value retrieved from the SMP, and not re-query it for every document. Since the SMP data is delivered over HTTP, one source of information for cache hints is the HTTP header data in the response, though it is unlikely that these are tailored for SMP data caching in all cases. It may be prudent to set a minimum and maximum cache time, regardless of HTTP headers.

(times are up for discussion)
5 minutes seems a reasonable minimum retention time. You should not cache SMP data for more than 24 hours.

### Error reporting

When contacting other service providers or a Peppol Authority to report issues about specific transactions or errors, include at the very least the following. This information is necessary to find information about the problem in log files.

- The exact time of the transaction (failure), including the time zone
- The sender ID
- The receiver ID
- The invoice ID
- If known: the transaction ID


## Report issues and request changes

If you have additional best practices, or want to suggest changes, please create a pull request for this repositry, create an issue on Github, or send an e-mail to [operations@peppolautoriteit.nl](mailto:operations@peppolautoriteit.nl?subject=[NPa%20GitHub]%20Best%20Practices%20change%20request) .
