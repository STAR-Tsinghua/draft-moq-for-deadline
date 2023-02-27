---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "MoQ relay for support of deadline-aware media transport"
abbrev: "MoQ relay for deadline"
category: info

docname: draft-mc-moq-relay-for-deadline-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date: {DATE}
consensus: true
v: 3
area: transport
workgroup: moq
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  github: "STAR-Tsinghua/DTP-draft"

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
  ins: H. Shi
  fullname: Hang Shi
  org: Huawei
  email: shihang9@huawei.com

- ins: W. Cao
  fullname: Wei Cao

normative:

informative:

--- abstract

This document defines MoQ relay's behavior to provide deliver-before-deadline transport. This memo intends to introduce deadline-aware operations to the MoQ relay to decrease end-to-end latency in real-time media transmission.

--- middle

# Introduction

Media over QUIC (MoQ) aims at building a system to better support real-time media transport like live streaming, online meeting, remote desktop, etc. These use cases usually require receiving their data before a certain time i.e. deadline. For example, a video conference application generally requires the end-to-end delay to be below human perception (about 100ms), to enable smooth interaction among participants. In such a system, the buffer will only hold the latest stream data and will drop the overdue data because the system will never use it.

Deadline-aware actions, such as passing deadline-related information from the endpoint to the relay and deadline-aware scheduling on relay nodes will increase the timeliness of data and decrease the cost of bandwidth. These mechanisms are tested in Deadline-aware Transport Protocol ({{!I-D.draft-shi-quic-dtp-07}}) and results show that deadline-aware actions can stop sending outdated data, prioritize urgent data and prevent useless re-transmission, decrease data queuing time, and increase punctuality of data. Deadline-aware scheduling can also provide a better overall user experience while serving clients with different delay requirements. For example, a relay may simultaneously forward a simple live stream and an online meeting stream. The live stream may tolerate a 1s delay while the online meeting only accepts a 100ms delay. It may be a good idea to forward the meeting stream first.

MoQ relays can benefit from deadline-aware actions and provide a better user experience. MoQ relays with deadline-aware actions can better schedule stream data to decrease queuing time, and prevent waste of re-transmission of overdue data to further decrease end-to-end delay. MoQ relays can also use deadline-related information to develop better data delivery strategies to increase overall user experience.

This document proposes Deadline-aware MoQ Relay to provide deliver-before-deadline transmission on MoQ relay nodes by using deadline-aware actions such as scheduling, data canceling, and redundancy coding. This relay design can act as an extension of a basic MoQ relay standard to provide better support to transmit media data in time.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Design Requirements

## Requirement of Relay Topology

Deadline-aware MoQ Relay SHOULD support multiple relay topologies like those mentioned in {{?I-D.draft-shi-moq-design-space-analysis-of-moq-00}}. The topology of the Deadline-aware MoQ Relay SHOULD NOT be specific.

## Requirement of MoQ Architecture

Deadline-aware MoQ Relay SHOULD support different MoQ architectures and SHOULD be able to cooperate with diverse MoQ implementations.

## Block-base Transport

Deadline-aware MoQ Relay SHOULD support block-based transport. A block is a basic, atomic data unit in the MoQ system that can be used independently at an endpoint like a video frame. The endpoint and relay SHOULD mark the boundary of a data block in some ways and inform the relays.

## Metadata

Deadline-aware MoQ Relay needs some metadata of the data block to enable deadline-aware actions. The endpoint and relay SHOULD attach the following metadata to each data block.

- size: the size of the data block in bytes
- priority: the global priority of the block in an unsigned integer
- deadline: the expected latency of the data. Relay is allowed to drop overdue data. Usually measured in milliseconds.
- start timestamp: the Unix timestamp when the block is created

Relays SHOULD keep track of the metadata of all the blocks it receives until the block misses its deadline.

Relays MAY synchronize the metadata of blocks with each other when necessary.

## Clock Synchronization

To support precise deadline-aware action, all the endpoints and relays SHOULD do clock synchronization. The time bias SHOULD be less than 1 ms.

# Deadline-aware Action of Relay

## Deadline-aware Scheduling and Cancelling

Deadline-aware MoQ Relay can use the information of cached data, block priority, deadline, and others to make stream-level scheduling. The scheduler SHOULD try to minimize the total time of queuing and try to meet the deadline requirements of as many blocks as possible.

In some cases where the data can be partially accepted, Deadline-aware MoQ Relay MAY cancel some blocks. The relays MAY inform the endpoints and other relays about the drop of these data blocks.

## Optional: Deadline-aware Redundancy Coding

Deadline-aware MoQ Relay MAY add redundancy data to blocks when the block is close to its deadline or the loss rate of the network increases. This helps to prevent the block from being re-transmitted and decreases latency. When the first relay encodes the data with redundancy coding, the following nodes may benefit from it.

If redundancy coding is enabled, every relay node SHOULD implement an encoder and decoder according to the redundancy coding method. The endpoint MAY implement the redundancy encoder and decoder to fully utilize the relay's redundancy coding function.

The first relay node that encodes the data SHOULD add redundancy-related information in the metadata of the data block. The last relay before the receiver endpoint should decode the data and forward it to the endpoint unless the endpoint can decode the data itself.

# Other Design Considerations

## Drop Notification

When a relay drops a data block due to missing of deadline or other reasons, it might be useful to send an explicit dropping message to other relays and the endpoint to inform them about an active loss of data.

## Data Buffer

The relay may buffer some data during forwarding. These data blocks may be used for re-transmission, serving users with larger delay tolerance and other usages. However, whether and how should we implement such a buffer still need further discussion.

# Security Considerations

The data block, metadata, and control message of the Deadlind-aware MoQ Relay should all be encrypted by QUIC's encryption method. The relays SHOULD be able to decrypt the metadata and relay-limited control message. The relays SHOULD NOT decrypt the content of the data block.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
