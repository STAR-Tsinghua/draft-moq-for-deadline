---
title: "MoQ relay for support of deadline-aware media transport"
abbrev: "MoQ relay for deadline"
category: info

docname: draft-ma-moq-relay-for-deadline-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date: {DATE}
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Media Over QUIC"
keyword:
  - deadline
  - relay
venue:
  github: "simonkorl/draft-moq-for-deadline"

author:
-
  ins: Y. Cui
  fullname: Yong Cui
  org: Tsinghua University
  street: 30 Shuangqing Rd
  city: Beijing
  country: China
  email: cuiyong@tsinghua.edu.cn

-
  ins: C. Ma
  fullname: Chuan Ma
  org: Tsinghua University
  street: 30 Shuangqing Rd
  city: Beijing
  country: China
  email: mc21@mails.tsinghua.edu.cn

-
  ins: Y. Liao
  fullname: Yixin Liao
  org: Tsinghua University
  street: 30 Shuangqing Rd
  city: Beijing
  country: China

-
  ins: H. Shi
  fullname: Hang Shi
  org: Huawei
  email: shihang9@huawei.com

normative:

informative:

--- abstract

This draft specifies the behavior of MoQ relays for delivering media before the deadline to decrease end-to-end latency and save transport costs in media transmission. To achieve this, the draft introduces deadline-aware actions prioritizing media streams with earlier deadlines, ensuring timely transmission while minimizing costs.

--- middle

# Introduction

Media over QUIC (MoQ) is a transport system designed to provide efficient media transport. However, some use cases, such as live streaming, online meetings, and gaming, require the client to receive their media before a specific time, referred to as the 'deadline.' Exceeding the deadline results in dropped data, which can increase latency and negatively affect user experience.

To address this issue, a deliver-before-deadline transport service can be provided, which is the goal of the Deadline-aware Transport Protocol (DTP) proposed in {{!I-D.draft-shi-quic-dtp-07}}. DTP leverages stream-level scheduling, active stream canceling, and redundancy coding to prioritize urgent data and prevent outdated data from blocking later data.

This document proposes the behavior of deadline-aware actions on MoQ relay nodes, extending the basic MoQ relay to provide deliver-before-deadline transmission. The relay design utilizes scheduling, data canceling, and redundancy coding to decrease queuing time, prevent unnecessary re-transmission of overdue data, and ultimately reduce end-to-end latency. By providing better data delivery strategies, MoQ relays with deadline-aware actions can significantly enhance overall user experience in media transport.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Design Requirements

## Support Different Relay Topology and Architecture

Deadline-aware MoQ Relay is recommended to support various relay topologies, as discussed in {{?I-D.draft-shi-moq-design-space-analysis-of-moq-00}}. Each relay topology may require a different architecture. Therefore, the deadline-aware feature should be designed as a plugin that can be easily implemented regardless of the topology and architecture.

## Support different MoQ transport implementations

Depending on the MoQ implementation, the block transmission may be mapped to different mechanisms of QUIC, such as matching a block to multiple QUIC datagrams or a single QUIC stream. It is recommended that Deadline-aware MoQ Relay support various MoQ transport implementations.

## Clock Synchronization

To enable accurate deadline-aware actions, it is recommended that all endpoints and relays perform clock synchronization.

# Design

## Block-base Transport

Deadline-aware MoQ Relay SHOULD support block-based transport to enable deadline-aware actions. A block is a basic data unit in the MoQ system like a video frame. A block can be reliably delivered, partially delivered or dropped, depending on the application logic. A block SHOULD contain a block ID in the block’s header.

When the relay receives data without deadline-related information from the endpoint, it MAY choose to convert the data into block type or forward it without utilizing any deadline-aware actions.

## Metadata for Deadline

For Deadline-aware MoQ Relay, data block metadata is required to enable deadline-aware actions. Both the endpoint and the relay SHOULD attach the following metadata to each data block when using deadline-aware actions:

- size: the size of the data block in bytes
- priority: the block's relative priority in a single session
- deadline: the expected completion timestamp of the block. The relay can drop overdue data.

The relay SHOULD maintain track of the metadata of a block until the block misses its deadline.

Additionally, if the endpoint does not offer metadata in the header of a data block, relays MAY implement other mechanisms to acquire and synchronize deadline-related metadata.

## Deadline-aware Action

### Deadline-aware Scheduling and Cancelling

When implementing deadline-aware actions, the Deadline-aware MoQ Relay can utilize the block metadata for scheduling blocks at the block-level. The scheduler SHOULD minimize the total time of queuing and try to meet the deadline requirements of as many high-priority blocks as possible.

If a block misses its deadline, Deadline-aware MoQ Relay MAY cancel it. In such cases, the endpoints SHOULD be able to accept partially received data and not request for data re-transmission when a block is dropped. Additionally, the relays MAY inform the endpoints and other relays about the cancellation of these blocks.

### Deadline-aware Redundancy Coding

To improve reliability and decrease latency, Deadline-aware MoQ Relay MAY introduce redundancy data to blocks close to their deadline or transmitted over a network with a high loss rate. This redundancy can help to prevent the need for re-transmission. If the first relay adds redundancy coding to the data, other relay nodes in the network may benefit from it.

When redundancy coding is enabled, at least two nodes SHOULD implement a pair of encoder and decoder that comply with the redundancy coding method. The endpoint MAY also implement a redundancy encoder and decoder to utilize the relay's redundancy coding function fully.

The first node that encodes the data with redundancy coding MUST add redundancy-related information to the metadata of the data block.

# Other Design Considerations

## Drop Notification

In situations where a data block is dropped by a relay due to a missed deadline or other reasons, sending an explicit dropping message to other relays and the endpoint can be useful to notify them of the loss of data. The dropping message may include information about the block, such as its ID and metadata.

## Data Buffer

Further discussion is required to determine if the relay should implement a buffer for data blocks during forwarding and how such a buffer should be implemented. This buffer may be used for re-transmission purposes and may benefit users with larger delay tolerance, among other potential uses.

# Security Considerations

Access to the metadata of the Deadline-aware MoQ Relay SHOULD be limited to selected relays. The relay SHOULD NOT access the content of the data block.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

We sincerely thank Wei Cao for his advice and revisions to this draft.
