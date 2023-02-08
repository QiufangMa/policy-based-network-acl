---
title: "A Policy-based Network Access Control"
abbrev: "A Policy-based NACL"
category: std

docname: draft-ma-opsawg-ucl-acl-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "OPSAWG"
keyword:
 - ACL
 - Policy-based
 - BYOD

author:
 -
    fullname: Qiufang Ma
    organization: Huawei
    street: 101 Software Avenue, Yuhua District
    city: Jiangsu
    code: 210012
    country: China
    email: maqiufang1@huawei.com
 -
    fullname: Qin Wu
    organization: Huawei
    street: 101 Software Avenue, Yuhua District
    city: Jiangsu
    code: 210012
    country: China
    email: bill.wu@huawei.com
 -
    fullname: Mohamed Boucadair
    organization: Orange
    city: Rennes
    code: 35000
    country: France
    email: mohamed.boucadair@orange.com
 -
    fullname: Daniel King
    organization: Old Dog Consulting
    country: United Kingdom
    email: daniel@olddog.co.uk

normative:

informative:
  NIST-ABAC:
     title: Guide to Attribute Based Access Control (ABAC) Definition and Considerations
     author:
       -
         fullname: Vincent C. Hu
     target: https://www.iana.org/assignments/media-types
     date: January 2014

  RADIUS-Types:
     title: RADIUS Types
     author:
       -
         organization: "IANA"
     target: http://www.iana.org/assignments/radius-types
     date: false

--- abstract

   This document defines a YANG module for policy-based network access
   control, which provide consistent and efficient enforcement of
   network access control policies based on group identity.  In
   addition, this document defines a mechanism to ease the maintenance
   of the mapping between a user-group ID and a set of IP/MAC addresses
   to enforce policy-based network access control.

   Also, the document defines a common schedule YANG module which is
   designed to be applicable for policy activation based on date and
   time conditions.

   In addition, the document defines a RADIUS attribute that is used to
   communicate the user group identifier as part of identification and
   authorization information.

--- middle

# Introduction

   With the increased adoption of remote access technologies (e.g.,
   Virtual Private Networks (VPNs)) and Bring Your Own Device (BYOD)
   policies, enterprises adopted more flexibility related to how, where,
   and when employees work and collaborate.  However, more flexibility
   comes with increased risks.  Enabling office flexibility (e.g.,
   mobility across many access locations) for large-scale employees
   induces a set of challenges compared to conventional network access
   management approaches.  Examples of such challenges are listed below:

   *  Endpoints do not have a stable IP address.  For example, Wireless
      LAN (WLAN) and VPN clients, as well as back-end Virtual Machine
      (VM)-based servers, can move; their IP addresses could change as a
      result.  This means that relying on IP/transport fields (e.g., the
      5-tuple) is inadequate to ensure consistent and efficient security
      policy enforcement.  IP address-based policies may not be flexible
      enough to accommodate endpoints with volatile IP addresses.

   *  With the massive adoption of teleworking, there is now a need to
      apply different security policies to the same set of users under
      different circumstances (e.g., prevent relaying attacks against a
      local attachment point to the Enterprise network).  For example,
      network access might be granted based upon criteria such as users'
      access location, source network reputation, users' role, time-of-
      day, type of network device used (e.g., corporate issued device
      versus personal device), device's security posture, etc.  This
      means the network needs to recognize the users' identity and their
      current context, and map the users to their correct access
      entitlement to the network.

   This document defines a common schedule YANG module which is designed
   to be applicable for policy activation based on date and time
   condition.  This model can be used in other scheduling contexts.

   The document also defines a YANG module for policy-based Network
   Access Control ({{NIST-ABAC}}), which extends the IETF Access Control
   Lists (ACLs) module defined in {{!RFC8519}}.  This module defined in
   {{NIST-ABAC}} is meant to ensure consistent enforcement of ACL policies
   based on group identity.  In addition, this document defines a
   mechanism to establish a mapping between the user-group ID and common
   IP packet headers and other enclosed packet data (e.g., MAC address)
   to execute the policy-based access control.

   Last, the document defines a RADIUS attribute that is used to
   communicate the user group identifier as part of identification and
   authorization information (Section 6).

   As the ACL notion has been generalized, not to be device-specific,
   but also be defined at network/administrative domain
   levels {{?I-D.dbb-netmod-acl}}, the YANG module for policy-based network
   access control defined in this document does not limit how it can be
   used.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

   The meanings of the symbols in tree diagrams are defined in
   [RFC8340].

   The document uses the terms defined in {{!RFC8519}}.

   In the current version of the draft, the term "endpoint" refers also
   to the host or end user that actually connect to a network.  Endpoint
   device refers to servers, IoTs and other devices owned by the
   enterprise.

#  Sample Usage

   Access to some networks (e.g., Enterprise Networks) require to
   recognize the users' identities no matter how, where, and when they
   connect to the network resources.  Then, the network maps the
   (conencting) users to their access authorization rights.  Such rights
   are defined following local policies.  As discussed in the
   introduction, because there is a large number of users and the source
   IP addresses of the same user are in different network segments,
   deploying a network access control policy for each IP address or
   network segment is heavy workload.  An alternative approach is to
   configure endpoint groups to classify users and enterprise devices
   and associate ACLs with endpoint groups so that endpoint in each
   group can share a group of ACL rules.  This approach greatly reduces
   the workload of the administrators and optimizes the ACL resources.
   The Network ACLs (NACLs) can be provisioned on devices using specific
   mechanisms, such as {{!RFC8519}} or {{?I-D.dbb-netmod-acl}}.

   Network access control policies may need to vary over time.  For
   example, companies may restrict/grant employees access to specific
   resources (internal and/or external resources) during work hours,
   while another policy is adopted during off-hours and weekends.  A
   network administrator may also require to enforce traffic shaping
   (Section 2.3.3.3 of {{?RFC2475}}) and policing (Section 2.3.3.4 of
   {{?RFC2475}}) during peak hours in order not to affect other data
   services.

#  Policy-based Network Access Control

##  Overview

   To provide real-time and consistent enforcement of access control
   policies, the following functional entities and interfaces are
   involved:

   *  A Service Orchestrator which coordinates the overall service,
      including security policies.  The service may be connectivity or
      any other resources that can be hosted and offerd by a newtork.

   *  The SDN Controller is responsible for maintaining endpoint-group
      based ACLs and mapping the endpoint-group to the associated
      attributes information (e.g., IP/MAC address).  The SDN Controller
      also works as the Policy Decision Point (PDP) {{?RFC3198}} and pushes
      the required access control policies to relevant PEPs (Policy
      Enforcement Points) that need them.  A PDP is also known as
      "policy server" {{?RFC2753}}.

      The SDN Controller may interacts with AAA server or NAS.

   *  A Network Access Server (NAS) entity which handles authentication
      requests.  The NAS interacts with an AAA server to complete user
      authentication using Remote Authentication Dial-in User Service
      (RADIUS) {{!RFC2865}}.  When access is granted, the AAA server
      provides the group identifier (group ID) to which the user belongs
      when the user first logs onto the network.  A new attribute is
      defined in Section 6.

   *  The Policy Enforcement Point (PEP) {{?RFC3198}} is the central entity
      which is responsible for enforcing appropriate access control
      policies.  In some cases, A PEP may map incoming packets to their
      associated source or destination endpoint-group IDs, and acts on
      the endpoint-group ID based ACL policies, e.g., a NAS as the PEP
      or a group identifier could be carried in packet header (see
      Section 6.2.3 in {{?I-D.ietf-nvo3-encap}}).  While in other cases,
      the SDN controller maps group ID to the related common packet
      header and delivers IP/MAC address based ACL policies to the
      required PEPs.

      Multiple PEPs can be involved in a network.

      A PEP exposes a NETCONF interface to the SDN Controller.

   {{figure-1}} provides the overall architecture and procedure for policy-
   based access control management.

~~~~
                                         +------------+
                                         |Orchestrator|
                                         +------+-----+
       Service                                  | (Step 1)
      ------------------------------------------+-------------
      ------------------------------------------+-------------
       Network                                  |
                                 (Step 4)       |
       +-------+        +--------+     +--------+-----------+
       |User #1+--+     | AAA    |     |    SDN Controller  |
       +-------+  |     | Server +-----+        (PDP)       |
                  |     +----+---+     +--------+-----------+
                  |          |                  |
                  |          |           +---------------+(Step 5)
        (Step 2)  |          |(Step 3)   |               |
                  |          |           |               |
                  |        +-+-----------+---------------+------------+
                  |        | +----------------------+   +------------+|
       +-------+  +--------+ | Network Access Server|   |firewall,etc||
       |User #2+-----------+ |       (NAS)          |   +------------+|
       +-------+           | +----------------------+                 |
                           |                     (PEP)                |
                           +------------------------------------------+
~~~~
{: #figure-1 title="An Architecture for Group-based Policy Management" artwork-align="center"}

   In reference to {{figure-1}}, the following typical flow is experienced:

   Step 1:  Administrators (or the Orchestrator) configure an SDN
      controller with network-level ACLs using the YANG module defined
      in Section 5.2.

   Step 2:  When a user first logs onto the network, the user is
      required to be authenticated (e.g., using user name and password)
      at the NAS.

   Step 3:  The authentication request is then relayed to the AAA server
      using RADIUS {{!RFC2865}}.  It is assumed that the AAA server has
      been appropriately configured to store user credentials, e.g.,
      user name, password, group information and other user attributes.

      If the authentication request succeeds, the user is placed in a
      user-group which is returned to the network access server as the
      authentication result (see Section 6).

      If the authentication fails, the user is not assigned a user-
      group, which also means that the user has no or limited access
      permissions for the network (as a fucntion of the local policy).
      ACLs are enforced so that flows from that IP address are discarded
      (or rate-limited) by the network.  In some implementations, AAA
      server can be integrated with an SDN controller.

   Step 4:  Either the AAA server or the NAS notify the SDN controller
      the mapping between the user-group ID and related common packet
      header attributes (e.g., IP/MAC address).

   Step 5:  The access control policies are maintained on relevant PEPs
      under the controller's management.

##  Endpoint Group

###  User Group

   The user-group ID is an identifier that represents the collective
   identity of a group of users.  It is determined by a set of
   predefined policy criteria (e.g., source IP address, geolocation
   data, time of day, or device certificate).  Users may be moved to
   different user-groups if their composite attributes, environment,
   and/or local enterprise policy change.

   A user is authenticated, and classified at the network ingress, and
   assigned to a user-group.  A user's group membership may change as
   aspects of the user change.  For example, if the user-group
   membership is determined solely by the source IP address, then a
   given user's user-group ID will change when the user moves to a new
   IP address that falls outside of the range of addresses of the
   previous user-group.

   This document does not make any assumption about how groups are
   defined.  Such considerations are deployment specific and are out of
   scope.  However, and for illustration purposes, Table 1 shows an
   example of how user-group definitions may be characterized.  User-
   groups may share several common criteria.  That is, user-group
   criteria are not mutually exclusive.  For example, the policy
   criteria of user-groups R&D Regular and R&D BYOD may share the same
   set of users that belong to the R&D organization, and differ only in
   the type of clients (firm-issued clients vs. users' personal
   clients).  Likewise, the same user may be assigned to different user-
   groups depending on the time of day or the type of day (e.g.,
   weekdays versus weekends), etc.

~~~~
       +--------------+------------+--------------------------------+
       |  Group Name  |  Group ID  |        Group Definition        |
       +--------------+------------+--------------------------------+
       |   R&D        |     10     |  R&D employees                 |
       +--------------+------------+--------------------------------+
       |   R&D BYOD   |     11     |  Personal devices of R&D       |
       |              |            |  employees                     |
       +--------------+------------+--------------------------------+
       |   Sales      |     20     |  Sales employees               |
       +--------------+------------+--------------------------------+
       |   VIP        |     30     |  VIP employees                 |
       +--------------+------------+--------------------------------+
       |   Workflow   |     40     |  IP addresses of Workflow      |
       |              |            |  resource servers              |
       +--------------+------------+--------------------------------+
       | R&D Resource |     50     | IP addresses of R&D resource   |
       |              |            | servers                        |
       +--------------+------------+--------------------------------+
       |Sales Resource|     54     | IP addresses of Sales resource |
       |              |            | servers                        |
       +--------------+------------+--------------------------------+

                   Table 1: User-Group Example
~~~~


###  Device Group

   The device-group ID is an identifier that represents the collective
   identity of a group of enterprise end devices.  An enterprise device
   could be an server that hosts applications or software that deliver
   services to enterprise users.  It could also be an enterprise IoT
   device that serve a limited purpose, e.g., a printer that allows
   users to scan, print and send emails.

   Users accessing to enterprise device should be strictly controlled.
   Matching abstract device group ID instead of specified addresses in
   ACL polices helps shield the consequences of address change (e.g.,
   back-end Virtual Machine (VM)-based server migration).

#  YANG Modules

##  The Schedule YANG Module

   This module defines a common schedule YANG module.  It is inspired
   from the "period of time" and "recurrence rule" format defined in
   {{?RFC5545}}.

   This module is defined as a standalone module rather than in the UCL
   module with the intention that the time/date definition can be
   reused.

###  Module Overview

   {{figure-3}} provides an overview of the tree structure of the "ietf-
   schedule" module.

~~~~
   module: ietf-schedule
     grouping period:
       +-- period-of-time
          +-- (forms)?
             +--:(period-explicit)
             |  +-- explicit-start?   yang:date-and-time
             |  +-- explicit-end?     yang:date-and-time
             +--:(period-start)
                +-- start?            yang:date-and-time
                +-- duration?         yang:time-no-zone
     grouping recurrence:
       +-- recurrence
          +-- freq           enumeration
          +-- (until-or-count)?
          |  +--:(until)
          |  |  +-- until?   union
          |  +--:(count)
          |     +-- count?   uint32
          +-- interval?      uint32
          +-- bysecond*      uint32
          +-- byminute*      uint32
          +-- byhour*        uint32
          +-- byday* [weekday]
          |  +-- direction?   int32
          |  +-- weekday?     schedule:weekday
          +-- bymonthday*    int32
          +-- byyearday*     int32
          +-- byyearweek*    int32
          +-- byyearmonth*   uint32
          +-- bysetpos*      int32
          +-- wkst*          schedule:weekday
~~~~
{: #figure-3 title="UCL Tree Diagram" artwork-align="center"}

###  The YANG Module

   This module imports types defined in {{!RFC6991}}.

~~~~
<CODE BEGINS>
file="ietf-schedule@2023-01-19.yang"
{::include ./yang/ietf-schedule.yang}
<CODE ENDS>
~~~~

##  The UCL Extension to the ACL Model

###  Module Overview

   {{figure-4}} provides the tree strcuture of the "ietf-ucl-acl" module.

~~~~
module: ietf-ucl-acl
  augment /acl:acls/acl:acl:
    +--rw endpoint-groups
       +--rw endpoint-group* [endpoint-group-id]
          +--rw endpoint-group-id     uint32
          +--rw (group-type)?
             +--:(user-group)
             |  +--rw user-group
             |     +--rw role?   string
             +--:(device-group)
                +--rw device-group
                   +--rw device-type?   string
  augment /acl:acls/acl:acl/acl:aces/acl:ace/acl:matches:
    +--rw endpoint-group {match-on-endpoint-group}?
       +--rw source-match
       |  +--rw endpoint-group-id?
       |          -> ../../../../../../endpoint-groups/endpoint-group/endpoint-group-id
       +--rw destination-match
          +--rw endpoint-group-id?
          |       -> ../../../../../../endpoint-groups/endpoint-group/endpoint-group-id
  augment /acl:acls/acl:acl/acl:aces/acl:ace:
    +--rw time-range {match-on-time}?
       +--rw (time-range-type)?
          +--:(periodic-range)
          |  +--rw recurrence
          |     +--rw freq           enumeration
          |     +--rw (until-or-count)?
          |     |  +--:(until)
          |     |  |  +--rw until?   union
          |     |  +--:(count)
          |     |     +--rw count?   uint32
          |     +--rw interval?      uint32
          |     +--rw bysecond*      uint32
          |     +--rw byminute*      uint32
          |     +--rw byhour*        uint32
          |     +--rw byday* [weekday]
          |     |  +--rw direction?   int32
          |     |  +--rw weekday      schedule:weekday
          |     +--rw bymonthday*    int32
          |     +--rw byyearday*     int32
          |     +--rw byyearweek*    int32
          |     +--rw byyearmonth*   uint32
          |     +--rw bysetpos*      int32
          |     +--rw wkst*          schedule:weekday
          +--:(absolute-range)
             +--rw period-of-time
                +--rw (forms)?
                   +--:(period-explicit)
                   |  +--rw explicit-start?   yang:date-and-time
                   |  +--rw explicit-end?     yang:date-and-time
                   +--:(period-start)
                      +--rw start?            yang:date-and-time
                      +--rw duration?         yang:time-no-zone
~~~~
{: #figure-4 title="UCL Extension" artwork-align="center"}

   This module specifies an extension to the IETF-ACL model {{!RFC8519}}
   such that the UCL group index may be referenced by augmenting the
   "matches" data node.

###  The YANG Module

   This module imports types defined in {{!RFC6991}}, {{!RFC8194}}, and
   {{!RFC8519}}.

~~~~
<CODE BEGINS>
file="ietf-ucl-acl@2023-01-19.yang"
{::include ./yang/ietf-ucl-acl.yang}
<CODE ENDS>
~~~~

# User Access Control Group ID RADIUS Attribute

   The User-Access-Group-ID RADIUS attribute and its embedded TLVs are
   defined with globally unique names.  The definition of the attribute
   follows the guidelines in Section 2.7.1 of {{!RFC6929}}.  This attribute
   is used to indicate the user-group ID to be used by the NAS.  When
   the User-Access-Group-ID RADIUS attribute is present in the RADIUS
   Access-Accept, the system applies the related access control to the
   users after it authenticates.

   The value fields of the Attribute are encoded in clear and not
   encrypted as, for example, Tunnel- Password Attribute {{?RFC2868}}.

   The User-Access-Group-ID Attribute is of type "string" as defined in
   Section 3.5 of {{!RFC8044}}.

   The User-Access-Group-ID Attribute MAY appear in a RADIUS Access-
   Accept packet.  It MAY also appear in a RADIUS Access-Request packet
   as a hint to the RADIUS server to indicate a preference.  However,
   the server is not required to honor such a preference.

   The User-Access-Group-ID Attribute MAY appear in a RADIUS CoA-Request
   packet.

   The User-Access-Group-ID Attribute MAY appear in a RADIUS Accounting-
   Request packet.

   The User-Access-Group-ID Attribute MUST NOT appear in any other
   RADIUS packet.

   The User-Access-Group-ID Attribute is structured as follows:

   Type

      241

   Length

      This field indicates the total length, in octets, of all fields of
      this attribute, including the Type, Length, Extended-Type, and the
      "Value".

   Extended-Type

      TBA1

   Value

      This field contains the user group ID.

   The User-Access-Group-ID Attribute is associated with the following
   identifier: 241.TBA1.

#  Table of Attributes

   The following table provides a guide as what type of RADIUS packets
   that may contain User-Access-Group-ID Attribute, and in what
   quantity.

~~~~
Access- Access- Access-  Challenge Acct.    #        Attribute
Request Accept  Reject             Request
 0+      0+      0        0         0+      241.TBA1 User-Access-Group-ID

CoA-Request CoA-ACK CoA-NACK #        Attribute
  0+          0       0      241.TBA2 User-Access-Group-ID

   The following table defines the meaning of the above table entries:

   0  This attribute MUST NOT be present in packet.
   0+ Zero or more instances of this attribute MAY be present in packet.
~~~~

# Security Considerations

##  YANG

   The YANG modules specified in this document defines schema for data
   that is designed to be accessed via network management protocols such
   as NETCONF {{!RFC6241}} or RESTCONF {{!RFC8040}}.  The lowest NETCONF layer
   is the secure transport layer, and the mandatory-to-implement secure
   transport is Secure Shell (SSH) {{!RFC6242}}.  The lowest RESTCONF layer
   is HTTPS, and the mandatory-to-implement secure transport is TLS
   {{!RFC8446}}.

   The Network Configuration Access Control Model (NACM) {{!RFC8341}}
   provides the means to restrict access for particular NETCONF or
   RESTCONF users to a preconfigured subset of all available NETCONF or
   RESTCONF protocol operations and content.

   There are a number of data nodes defined in this YANG module that are
   writable, creatable, and deletable (i.e., config true, which is the
   default).  These data nodes may be considered sensitive or vulnerable
   in some network environments.  Write operations to these data nodes
   could have a negative effect on network and security operations.

   <<add-more-about privacy considerations as the modules manipulate PII
   data.>>

##  RADIUS

   RADIUS-related security considerations are discussed in {{!RFC2865}}.

   This document targets deployments where a trusted relationship is in
   place between the RADIUS client and server with communication
   optionally secured by IPsec or Transport Layer Security (TLS)
   {{?RFC6614}}.

#  IANA Considerations

##  YANG

   This document registers the following URI in the "IETF XML Registry" {{!RFC3688}}.

~~~~
        URI: urn:ietf:params:xml:ns:yang:ietf-ucl-group
        Registrant Contact: The IESG.
        XML: N/A, the requested URI is an XML namespace.

        URI: urn:ietf:params:xml:ns:yang:ietf-ucl-acl
        Registrant Contact: The IESG.
        XML: N/A, the requested URI is an XML namespace.
~~~~

   This document registers a YANG module in the "YANG Module Names"
   registry {{!RFC6020}}.

~~~~
        name:               ietf-ucl-group
        namespace:          urn:ietf:params:xml:ns:yang:ietf-ucl-group
        prefix:             uclg
        maintained by IANA: N
        reference:          RFC XXXX

        name:               ietf-ucl-acl
        namespace:          urn:ietf:params:xml:ns:yang:ietf-ucl-acl
        prefix:             uacl
        maintained by IANA: N
        reference:          RFC XXXX
~~~~

##  RADIUS

   IANA is requested to assign a new RADIUS attribute typs from the IANA
   registry "Radius Attribute Types" {{RADIUS-Types}}:

~~~~
      +==========+======================+===========+===============+
      | Value    | Description          | Data Type | Reference     |
      +==========+======================+===========+===============+
      | 241.TBA1 | User-Access-Group-ID | string    | This-Document |
      +----------+----------------------+-----------+---------------+

                         Table 2: RADIUS Attribute
~~~~

--- back

# Changes between Revisions

   v00 - v01
   *  Define a common schedule yang module and reuse in UCL yang module
      to support time/date-based activation condition.

   *  Either group-based or address-based ACL policies could be enforced
      at PEP, and allow group-based ACL policies maintained at the
      network controller.

   *  Optimize the process in section 4.1.

   *  Extend ACL module to support a generalized endpoint-group to cover
      both end users (e.g., enterprise employees) and enterprise hosts
      (e.g., IoT devices or servers);

   *  Simplify the definition of group in UCL model with only the most
      necessary group ID retained.

# Acknowledgments
{:numbered="false"}

   This work has benefited from the discussions of User-group-based
   Security Policy over the years.  In particular, [I-D.you-i2nsf-user-
   group-based-policy] and [I-D.yizhou-anima-ip-to-access-control-
   groups] provide mechanisms to establish the mapping between the IP
   address/prefix of user and access control group ID.

   Jianjie You, Myo Zarny, Christian Jacquenet, Mohamed Boucadair, and
   Yizhou Li contributed to an earlier version of [I-D.you-i2nsf-user-
   group-based-policy].  We would like to thank the authors of that
   draft on modern network access control mechanisms for material that
   assisted in thinking about this document.

   The authors would like to thank Joe Clarke, Bill Fenner, Benoit
   Claise, Rob Wiltion, David Somers-Harris for their valuable comments
   and great input to this work.