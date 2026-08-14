---
title: "A YANG Data Model and RADIUS Extension for Policy-Based Network Access Control"
abbrev: "A Policy-based Network Access Control"
category: std

docname: draft-ietf-opsawg-ucl-acl-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "OPSAWG"
keyword:
 - ACL
 - BYOD
 - Access control
 - Policy Enforcement Point
 - PEP
 - Policy enforcement
 - Policy Decision Point
 - PDP
 - Network Management
 - Service Provisioning
 - Group-Based Policy
 - GBP
 - Software-Defined Networking
 - SDN

author:
 -
    fullname: Qiufang Ma
    organization: Huawei
    role: editor
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
    role: editor
    city: Rennes
    code: 35000
    country: France
    email: mohamed.boucadair@orange.com
 -
    fullname: Daniel King
    organization: Lancaster University
    country: United Kingdom
    email: d.king@lancaster.ac.uk

normative:

informative:

  RADIUS-Types:
     title: RADIUS Types
     author:
       -
         organization: "IANA"
     target: https://www.iana.org/assignments/radius-types
     date: false

--- abstract

   This document defines a YANG data model for policy-based network access
   control, which provides enforcement of
   network access control policies based on group identity. This YANG data model extends Access Control Lists (ACLs) with date and time parameters to support schedule-aware policy enforcement.

   Specifically in scenarios where network access is triggered by user authentication, this document defines a mechanism to ease the maintenance
   of the mapping between a user group identifier and a set of packet header fields
   to enforce policy-based network access control. Moreover, the document defines a Remote Authentication Dial-in
   User Service (RADIUS) attribute that is used to
   communicate the user group identifier as part of identification and
   authorization information.

--- middle

# Introduction {#intro}

   With the increased adoption of remote access technologies (e.g.,
   Virtual Private Networks (VPNs) and Bring Your Own Device (BYOD)
   policies), enterprises adopted more flexibility related to how, where,
   and when employees work and collaborate.  However, more flexibility
   comes with increased risks.  Enabling office flexibility (e.g.,
   mobility across many access locations) introduces a set of challenges
   for large-scale enterprises compared to conventional network access management approaches.
   Examples of such challenges are listed below:

   *  Endpoints do not have stable and unique IP addresses.  For example, Wireless
      LAN (WLAN) and VPN clients, as well as back-end Virtual Machine
      (VM)-based servers, can move; their IP addresses could change as a
      result. Furthermore, mechanisms such as IPv6 temporary addresses {{?RFC8981}}, and Network Address Port Translation (NAPT) {{?RFC3022}} may further contribute to address instability and non-uniqueness. This complicates the consistent and efficient access control policy enforcement relying on IP/transport fields (e.g., the
      5-tuple). IP address-based policies may not be flexible
      enough to accommodate endpoints with volatile IP addresses.

   *  With the massive adoption of teleworking, there is a need to
      apply different security policies to the same set of endpoints under
      different circumstances (e.g., prevent relay attacks against a
      local attachment point to the enterprise network).  For example,
      network access might be granted based upon criteria such as users'
      access location, source network reputation, users' role, time-of-
      day, type of network device used (e.g., corporate-issued device
      versus personal device), device's security posture, etc. This
      means that the network needs to recognize the endpoints' identity and their
      current context, and map the endpoints to their correct access
      grant to the network.

   This document defines a YANG data model ({{sec-UCL}}) for policy-based network access control,
   which extends the IETF Access Control Lists (ACLs) module defined in {{!RFC8519}}.
   This module can be used to ensure consistent enforcement of ACL policies
   based on the group identity. Additionally, the YANG data model defined in the document also extends ACLs with date and time parameters to support schedule-aware policy enforcement.

   The ACL concept has been generalized to be device-nonspecific, and can be
   defined at network/administrative domain level {{?RFC9899}}. To
   allow for all  ACL applications, the YANG module for policy-based network
   ACL defined in {{sec-UCL}} does not limit how it can be used.

   Specifically in scenarios where network access is triggered by user authentication, this document also defines a mechanism to establish a mapping between (1) the
   user group identifier (ID) and (2) common IP packet header fields and other
   encapsulating packet data (e.g., MAC address) to execute the policy-based access control.
   Additionally, the document defines a Remote Authentication Dial-in
   User Service (RADIUS) {{!RFC2865}} attribute that is used to
   communicate the user group identifier as part of identification and
   authorization information ({{sec-radius}}).

   Although the document cites MAC addresses as an example in some sections, the
   document does not make assumptions about which identifiers are used to trigger ACLs.
   These examples should not be considered as recommendations. Readers should be
   aware that MAC-based ACLs can be bypassed by clearing the MAC address.
   Other implications related to the change of MAC addresses are discussed in
   {{?RFC9797}}.

  The document does not specify how to map the policy group identifiers to
dedicated packet fields. Group-Based Policy (GBP), discussed in {{Section 6.2.3 of ?RFC9638}},
   provides an example of how that may be achieved.

## Editorial Note (To be removed by RFC Editor)

   Note to the RFC Editor: This section is to be removed prior to publication.

   This document contains placeholder values that need to be replaced with finalized
   values at the time of publication.  This note summarizes all of the
   substitutions that are needed.  No other RFC Editor instructions are specified
   elsewhere in this document.

   Please apply the following replacements:

   * XXXX --> the assigned RFC number for this document
   * 2026-01-12 --> the actual date of the publication of this document

# Conventions and Definitions

{::boilerplate bcp14-tagged}

   The meanings of the symbols in tree diagrams are defined in
   {{?RFC8340}}.

   The document uses the following terms defined in {{!RFC8519}}:

   * Access Control Entry (ACE)

   * Access Control List (ACL)

   The following definitions are used throughout this document:

  Enterprise device:
  : A device that falls under the access control domain of
   centrally managed authority (enterprise administrator, typically).
   An enterprise device provides compute, memory, storage, and networking capabilities and connects to a network.
  : An enterprise device could be a server that hosts applications or software that deliver services to enterprise users. It could also be an enterprise Internet of Things (IoT) device that serves a limited purpose (e.g., a printer that allows users to scan and print), etc.
  : While a personal device (BYOD) is not a physical asset of the enterprise, it is subject to the enterprise' access control policies when accessing the enterprise resources controlled by the centrally managed authority.

   Endpoint:
   : Refers to an entity which could be an end-user, enterprise device, or application that actually connects to a network.

   Endpoint group:
   : Refers to a group of endpoints that share common access control policies.

   User group:
   : A group of end-users who will be assigned the same network access policy. An end-user is defined as a person. Refer to {{sec-ug}} for more details.

   device group:
   : A collection of enterprise devices that share a common access control policies. Refer to {{sec-dg}} for more details.

   Application group:
   : A collection of applications that share a common access control policies. An application is a software program used for a specific service. Refer to {{sec-ag}} for more details.

   Endpoint group identifier:
   : An identifier used to represent the collective identity of
   an endpoint group. An endpoint group may include a user group, device group, or application group.

   User group based Control List (UCL) data model:
   : A YANG data model for policy-based network access
     control that specifies an extension to the "ietf-access-control-list" module {{!RFC8519}}.
     It allows policy enforcement based on a group identifier, which can be used
     both at the network device level and at the network/administrative domain level.

   Policy:
   : A set of rules to administer, manage, and control access to network resources {{?RFC3198}}.

#  Sample Usage

   Access to some networks (e.g., enterprise networks) requires
   recognizing the endpoints' identities no matter how, where, and when they
   connect to the network resources.  Then, the network maps the
   (connecting) endpoints to their access authorization rights.  Such rights
   are defined using local policies.  As discussed in {{intro}},
   because (1) there is a large number of connecting endpoints and (2) an endpoint may have different
   source IP addresses in different network segments,
   deploying a network access control policy for each IP address or
   network segment requires a high overhead. An alternate approach is to configure endpoint groups to classify users,
   enterprise devices, and applications, and to associate ACLs with endpoint
   groups so that endpoints in each group can share a group of ACL rules.
   This approach greatly reduces
   the overhead of the administrators and optimizes the ACL resources.

   The network ACLs can be provisioned on devices using specific
   mechanisms, such as {{!RFC8519}} or {{?RFC9899}}.

   Different policies may need to be applied in different contextual situations.
   For example, companies may restrict (or grant) employees access to specific
   internal or external resources during work hours,
   while another policy is adopted during off-hours and weekends.  A
   network administrator may also require traffic shaping
   ({{Section 2.3.3.3 of ?RFC2475}}) and policing
   ({{Section 2.3.3.4 of ?RFC2475}}) during peak hours in order to not affect other data
   services.

#  Policy-based Network Access Control

##  Overview {#overview}

   An example architecture of a system that provides real-time and
   consistent enforcement of access control policies is shown in
   {{arch}}.  This architecture illustrates a user-centric flow, which
   includes the following functional entities and interfaces:

* A service orchestrator which coordinates the overall service, including security policies. The service may be connectivity or any other access to resources that can be hosted and offered by a network.

* A Software-Defined Networking (SDN) {{?RFC7149}} {{?RFC7426}} controller which is responsible for maintaining endpoint-group based ACLs and mapping the endpoint group to the associated attributes information (e.g., packet header fields). An SDN controller also behaves as a Policy Decision Point (PDP) {{?RFC3198}} and pushes the required access control policies to relevant Policy Enforcement Points (PEPs) {{?RFC3198}}. A PDP is also known as "policy server" {{?RFC2753}}.

    An SDN controller may interact with an Authentication, Authorization, and Accounting (AAA) {{?RFC3539}} server or a Network Access Server (NAS) {{?RFC7542}}.

* A NAS entity which handles authentication requests. The NAS interacts with an AAA server to complete user authentication using protocols like RADIUS {{!RFC2865}}. When access is granted, the AAA server provides the group identifier (group ID) to which the user belongs when the user first logs onto the network.

    A new RADIUS attribute is defined in {{sec-radius}} for this purpose.

* The AAA server provides a collection of authentication, authorization, and accounting functions. The AAA server is responsible for centralized user information management. The AAA server is preconfigured with user credentials (e.g., username and password), possible group identities and related user attributes (users may be divided into different groups based on different user attributes).

* The Policy Enforcement Point (PEP) is the central entity which is responsible for enforcing appropriate access control policies. A first deployment scenario assumes that the SDN controller maps the group ID to the related common packet header and delivers packet header fields based ACL policies to the required PEPs. Another deployment scenario may require that PEPs map incoming packets to their associated source and/or destination endpoint-group IDs, and acts upon the endpoint-group ID-based ACL policies (e.g., a group identifier may be carried in packet headers such as discussed in {{Section 6.2.3 of ?RFC9638}}).

    Multiple PEPs may be involved in a network.

    A PEP exposes a YANG-based interface (e.g., NETCONF {{?RFC6241}}) to an SDN controller.

{{arch}} provides the overall architecture and procedure for policy-based access control management.

~~~~ aasvg
{::include-fold ./examples/arch.txt}
~~~~
{: #arch title="An Example Architecture for User Group-based Policy Management" artwork-align="center"}

In reference to {{arch}}, the following typical flow is experienced:

Step 1:
:  Administrators (or a service orchestrator) configure an SDN
   controller with network-level ACLs using the YANG module defined
   in {{sec-UCL}}. An example is provided in {{controller-ucl}}.

Step 2:
:  When a user first logs onto the network, they are
      required to be authenticated (e.g., using username and password)
      at the NAS.

Step 3:
:  The authentication request is then relayed to the AAA server
      using a protocol such as RADIUS {{!RFC2865}}. It is assumed that the
      AAA server has been appropriately configured to store user credentials,
      e.g., username, password, group information, and other user attributes.
      This document does not restrict what authentication method is used. Administrators
      may refer to, e.g., {{Section 7.4 of ?I-D.ietf-radext-deprecating-radius}}
      for authentication method recommendations.
: If the authentication request succeeds, the user is placed in a
      user group with the identifier returned to the NAS
      as the authentication result (see {{sec-radius}}).
      If the authentication fails, the user is not assigned any user
      group, which also means that the user has no access (i.e., Access-Reject returned); or the user
      is assigned a special group with very limited access permissions
      for the network (as a function of the local policy). ACLs are
      enforced so that flows from that IP address are discarded
      (or rate-limited) by the network.
: In some implementations, the AAA server can be integrated with an SDN controller.

Step 4:
:  Either the AAA server or the NAS notifies an SDN controller
      of the mapping between the user group ID and related common packet
      header attributes (e.g., the 5-tuple). The exact details of how such notification is performed are out scope of this specification.

Step 5:
:  Either group-based or packet header field-based access control policies
      are maintained on relevant PEPs under the SDN controller's management.
      Both types of ACL policy may exist on
      the PEP. {{PEP-ucl}} and {{PEP-acl}} elaborate on each case.

A similar flow applies to policy management based on other endpoint group types, such as device or application groups,
except that the mapping between the group ID and related common packet
header attributes (e.g., 5-tuple) may be maintained on the SDN controller based on an inventory or an application registry. Particularly, the use of RADIUS exchanges is not required in such cases ({{sec-radius}}).

{{implement-considerations}} provides additional operational considerations.

##  Endpoint Group

###  User Group {#sec-ug}

   The user group is determined by a set of predefined policy criteria
   (e.g., source IP address, geolocation data, time of day, or device certificate).
   It uses an identifier (user group ID) to represent the collective identity of
   a group of users. Users may be moved to different user groups if their
   composite attributes, environment, and/or local enterprise policy change.

   A user is authenticated, and classified at the AAA server, and
   assigned to a user group.  A user's group membership may change as
   aspects of the user change.  For example, if the user group
   membership is determined solely by the source IP address, then a
   given user's group ID will change when the user is assigned a new
   IP address that falls outside of the range of addresses of the
   previous user group.

   This document does not make any assumption about how user groups are
   defined.  Such considerations are deployment-specific and are out of
   scope.  However, and for illustration purposes, {{ug-example}} shows
   an example of how user group definitions may be characterized. User
   groups may share several common criteria.  That is, user group
   criteria are not mutually exclusive.  For example, the policy
   criteria of user groups R&D Regular and R&D BYOD may share the same
   set of users that belong to the R&D organization, and differ only in
   the type of clients (firm-issued clients vs. users' personal
   clients).  Likewise, the same user may be assigned to different user
   groups depending on the time of day or the type of day (e.g.,
   weekdays versus weekends), etc.

| Group Name | Group ID | Group Description |
| R&D Regular|   foo-10 |  R&D employees                 |
| R&D BYOD   |   foo-11 |  Personal devices of R&D employees |
| Sales      |   foo-20 |  Sales employees               |
| VIP        |   foo-30 |  VIP employees                 |
{: #ug-example title='User Group Examples'}

###  Device Group {#sec-dg}

   The device group ID is an identifier that represents the collective
   identity of a group of enterprise devices.
   {{dg-example}} shows an example
   of how device group definitions may be characterized.

   | Group Name | Group ID | Group Description |
   | Workflow   |   bar-40     |  Workflow  resource servers   |
   | R&D Resource |   bar-50     | R&D resource servers |
   |Printer Resource|   bar-60     | Printer resources |
   {: #dg-example title='Device Group Example'}

   Matching abstract device group ID instead of specified addresses in
   ACL polices helps shield the consequences of address change (e.g.,
   back-end VM-based server migration).

### Application Group {#sec-ag}

   An application group is a collection of applications that share a common access control policies.
   A device may run multiple applications, and different policies might need to be
   applied to the applications and device. A single application may need to run on
   multiple devices/VMs/containers, the abstraction of application group eases the
   process of application migration. For example, the policy does not depend on the transport coordinates (i.e., 5-tuple).
   {{ag-example}} shows an example of how application group definitions may be characterized.

   | Group Name | Group ID | Group Description |
   | Audio/Video Streaming  |   baz-70   |  Audio/Video conferencing application |
   | Instant Messaging |   baz-80   | Messaging application |
   | document Collaboration |  baz-90  | Real-time document editing application |
   {: #ag-example title='Application Group Examples'}

## Relations Between Different Endpoint Groups

  Policy enforcement can be targeted to different endpoint groups in different scenarios.
  For example, when a user connects to the network and accesses an application hosted on one or multiple devices, access policies may be applied to different user groups.
  In some cases, applications and devices may operate and run without requiring any user interventions,
  or they may require user authentication but access rules do not differentiate between different users.
  This enables policies to be applied to the application or device group.
  A device group can be used when there is only one single application running on the device
  or different applications running but with the same access control rules.
  If there is an application running on different devices/VMs/containers, it is simpler
  to apply a single policy to the application group.

# The UCL Extension to the ACL Module

##   Module Overview

   This module specifies an extension to the "ietf-access-control-list" module {{!RFC8519}}. This extension adds
   endpoint groups so that an endpoint group identifier can be matched upon, and also
   enable access control policy activation based on date and time conditions.

   {{ucl-tree}} provides the tree structure of the "ietf-ucl-acl" module.

~~~~
{::include ./yang/ietf-ucl-acl-tree.txt}
~~~~
{: #ucl-tree title="UCL Extension" artwork-align="center"}

   The first part of the "ietf-ucl-acl" module augments the "acl" list in the
   "ietf-access-control-list" module {{!RFC8519}} with an "endpoint-groups" container
   having a list of "endpoint group" inside, each entry has a "group-id" that uniquely
   identifies the endpoint group and a "group-type" parameter to specify the endpoint group type.

> "group-id" is defined as a string rather than unsigned integer (e.g., uint32) to accommodate deployments which require some identification hierarchy within a domain. Such a hierarchy is meant to ease coordination within an administrative domain. There might be cases where a domain needs to tag packets with the group they belong to. The tagging does not need to mirror exactly the "group ID" used to populate the policy. How the "group-id" string is mapped to the tagging or field in the packet header in encapsulation scenario is outside the scope of this document. Augmentation may be considered in the future to cover encapsulation considerations.

   The second part of the "ietf-ucl-acl" module augments the "matches" container in the
   "ietf-access-control-list" module {{!RFC8519}} so that a source and/or destination endpoint group index
   can be referenced as the match criteria.

   The third part of the module augments the "ace" list in the "ietf-access-control-list"
   module {{!RFC8519}} with date and time specific parameters to allow ACE to be
   activated based on a date/time condition. Two types of time range are defined,
   which reuse "recurrence" and "period" groupings defined in the "ietf-schedule"
   YANG module in {{!RFC9922}}, respectively.

##  The "ietf-ucl-acl" YANG Module {#sec-UCL}

   This module imports types and groupings defined in the "ietf-schedule" module
   {{!RFC9922}}. It also augments the "ietf-access-control-list" module ({{Section 4.1 of !RFC8519}}).

~~~~ yang
<CODE BEGINS> file "ietf-ucl-acl@2026-01-12.yang"
{::include ./yang/ietf-ucl-acl.yang}
<CODE ENDS>
~~~~

# User Access Control Group ID RADIUS Attribute {#sec-radius}

This section defines a User-Access-Group-ID RADIUS attribute which is designed for user-centric access control scenarios where network access is triggered by user authentication and used to indicate the user group ID to be used by the NAS.
For other endpoint group types, such as device group or application group, the identifiers are typically pre-provisioned
on the SDN controller based on an inventory or an application registry.

The definition of the attribute
follows the guidelines in {{Section 2.7.1 of !RFC6929}}. When
the User-Access-Group-ID RADIUS attribute is present in the RADIUS
Access-Accept, the system applies the related access control to the
users after the user authenticates.

The User-Access-Group-ID Attribute is of type "string" as defined in
{{Section 3.5 of !RFC8044}}.

The User-Access-Group-ID Attribute MAY appear in a RADIUS
Access-Accept packet.  It MAY also appear in a RADIUS Access-Request packet
as a hint to the RADIUS server to indicate a preference.  However,
the server is not required to honor such a preference. If more than
one instance of the User-Access-Group-ID Attribute appears in a RADIUS
Access-Accept packet, this means that the user is a member of many groups.

The User-Access-Group-ID Attribute MAY appear in a RADIUS CoA-Request
packet.

The User-Access-Group-ID Attribute MAY appear in a RADIUS Accounting-Request
packet. Specifically, this may be used by a NAS to acknowledge that the attribute
was received in the RADIUS Access-Request and the NAS is enforcing that policy.

The User-Access-Group-ID Attribute MUST NOT appear in any other
RADIUS packet.

The User-Access-Group-ID Attribute is structured as follows:

   {: vspace="0"}
   Type
   : TBA1

   Length
   : This field indicates the total length, in octets, of all fields of
   this attribute, including the Type, Length, Extended-Type, and Value.
   : Length MUST be at least 4, and MUST NOT be more than 67.
   : The maximum length is 67 octets to accommodate the maximum group ID of 64 octets, plus one octet each for Type, Length, and Extended-Type.

   Data Type
   : string ({{Section 3.5 of !RFC8044}})

   Value
   : This field contains the user group ID.

#  Table of Attributes

   {{rad-att}} provides a guide as what type of RADIUS packets
   that may contain User-Access-Group-ID Attribute, and in what
   quantity.

|Access-Request	|Access-Accept	|Access-Reject	|Access-Challenge	| Attribute     |
| 0+            |  0+          | 0            |    0     | User-Access-Group-ID     |
|Accounting-Request|	CoA-Request|	CoA-ACK	|CoA-NACK		| Attribute     |
|    0+            | 0+         | 0       | 0        | User-Access-Group-ID     |
{: #rad-att title='Table of Attributes'}

Notation for {{rad-att}}:

   0
   :  This attribute MUST NOT be present in packet.

   0+
   : Zero or more instances of this attribute MAY be present in packet.

# Operational Considerations {#implement-considerations}

## Deployment Options

   The UCL model can be implemented in different ways.

   In some cases, the UCL data model is implemented at the network/administrative domain
   level with an SDN controller maintaining the dynamical mapping from endpoint
   group ID to IP/transport fields (e.g., the 5-tuple) and programing the PEPs with
   IP address/5-tuple based ACLs. In such cases, PEPs do not require implementing
   specific logic (including hardware) compared to the enforcement of conventional ACLs.

   It is possible for the UCL data model to be implemented at the device level.
   While it eliminates the need for an SDN controller to interact frequently
   with the PEPs for reasons like the user's context of network connection change
   or VM/application migration, dedicated hardware/software support might be needed
   for PEPs to understand the endpoint group identifier. In scenarios where the NAS
   behaves as the PEP which acquires the source and/or destination endpoint group
   ID from the AAA server, ACL policy enforcement based on the group identity without
   being encapsulated into packet headers might affect the forwarding performance.
   Implementations need to evaluate the operational tradeoff (flexibility brought
   to the network vs. the complexity of implementation) carefully. Such assessment
   is out of scope for this document.

## Hardware/Software Implications

   Some devices may not have built-in capabilities to enforce group-based match policies.
   Hardware or software upgrades may be required to support such feature by involved PEPs.

## Mapping Consistency

   The specification requires that adequate setup is put in place to map a Group ID to packet
   fields, typically managed by a controller. Special care should be taken
   to ensure that such mapping is appropriately enforced when distinct
   mechanisms (RADIUS, etc.) are supported in network.


# Security Considerations

##  YANG

   This section is modeled after the template described in {{Section 3.7.1 of ?RFC9907}}.

   The "ietf-ucl-acl" YANG module defines a data model
   that is designed to be accessed via YANG-based management protocols such
   as the Network Configuration Protocol (NETCONF) {{?RFC6241}} and RESTCONF {{?RFC8040}}. These YANG-based management
   protocols (1) have to use a secure transport layer (e.g., Secure Shell (SSH) {{?RFC4252}}, TLS {{?I-D.ietf-tls-rfc8446bis}}, and
   QUIC {{?RFC9000}}) and (2) have to use mutual authentication.

   The Network Configuration Access Control Model (NACM) {{!RFC8341}}
   provides the means to restrict access for particular NETCONF or
   RESTCONF users to a preconfigured subset of all available NETCONF or
   RESTCONF protocol operations and content.

   There are a number of data nodes defined in this YANG module that are
   writable/creatable/deletable (i.e., "config true", which is the
   default).  All writable data nodes are likely to be sensitive or
   vulnerable in some network environments.  Write
   operations (e.g., edit-config) and delete operations to these data
   nodes without proper protection or authentication can have a negative
   effect on network operations.  The following subtrees and data nodes
   have particular sensitivities/vulnerabilities:

   * /acl:acls/ucl:endpoint-groups/ucl:endpoint-group:
   : This list specifies all the endpoint group entries. Unauthorized write access to this
     list can allow intruders to modify the entries so as to forge an endpoint
     group that does not exist or maliciously delete an existing endpoint group,
     which could be used to craft an attack.

   * /acl:acls/acl:acl/acl:aces/acl:ace/acl:matches/ucl:endpoint-group:
   : This subtree specifies a source and/or endpoint group index as match criteria in the
     ACEs. Unauthorized write access to this data node may allow intruders to
     modify the group identity so as to permit access that should not be
     permitted, or deny access that should be permitted.

   * /acl:acls/acl:acl/acl:aces/acl:ace/ucl:effective-schedule:
   : It specifies the secheduling of ACLs. Unauthorized write access to this data node may allow intruders to
     alter it. This may lead to service disruption or unavailability. Strict access control must be implemented for write operations on this subtree to ensure that only authorized users can modify it.

   Some of the readable data nodes in this YANG module may be considered
   sensitive or vulnerable in some network environments.  It is thus
   important to control read access (e.g., via get, get-config, or
   notification) to these data nodes. Specifically, the following
   subtrees and data nodes have particular sensitivities/
   vulnerabilities:

   * /acl:acls/acl:acl/acl:aces/acl:ace/ucl:effective-schedule:
   : It specifies when the access control entry rules are applied. An
     unauthorized read access of the list will allow the attacker to determine
     which rules are applied, to better craft an attack.

##  RADIUS

   RADIUS-related security considerations are discussed in {{!RFC2865}}.
   An effort to deprecating insecure practices in RADIUS is provided in {{?I-D.ietf-radext-deprecating-radius}}.

   This document targets deployments where a trusted relationship is in
   place between the RADIUS client and server with communication
   optionally secured by IPsec or Transport Layer Security (TLS)
   {{?RFC6614}}{{?I-D.ietf-radext-radiusdtls-bis}}.

#  IANA Considerations

##  YANG

   This document registers the following URIs in the "IETF XML Registry" {{!RFC3688}}.

~~~~
        URI: urn:ietf:params:xml:ns:yang:ietf-ucl-acl
        Registrant Contact: The IESG.
        XML: N/A, the requested URI is an XML namespace.
~~~~

   This document registers the following YANG modules in the "YANG Module Names"
   registry {{!RFC6020}}.

~~~~
        name:               ietf-ucl-acl
        prefix:             ucl
        namespace:          urn:ietf:params:xml:ns:yang:ietf-ucl-acl
        maintained by IANA? N
        reference:          RFC XXXX
~~~~

##  RADIUS

   This document requests IANA to assign a new RADIUS attribute type in the 241-245 range from the IANA
   registry "Radius Attribute Types" {{RADIUS-Types}}:

| Value    | Description          | Data Type | Reference     |
| TBA1 | User-Access-Group-ID | string    | This-Document |
{: #rad-reg title='RADIUS Attribute'}


--- back

# Examples Usage

## Configuring the Controller Using Group based ACL {#controller-ucl}

   Let's consider an organization that would like to manage the access of R&D
   employees that bring personally owned devices (BYOD) into the workplace.

   The access requirements are as follows:

   * Permit traffic from R&D BYOD of employees, destined to R&D employees'
     devices every work day from 8:00:00 to 18:00:00 UTC, starting in January 1st, 2026.

   * Deny traffic from R&D BYOD of employees, destined to finance servers
     located in the enterprise DC network starting at 8:30:00 of January 20,
     2026 with an offset of -08:00 from UTC (Pacific Standard Time) and ending
     at 18:00:00 in Pacific Standard Time on December 31, 2026.

   The example shown in {{ex-controller-ucl}} illustrates the configuration of an SDN controller
   using the group-based ACL:

~~~~ json
{::include-fold ./examples/valid-controller-ucl.json}
~~~~
{: #ex-controller-ucl title="Example of UCL Configuration"}

## Configuring a PEP Using Group-based ACL {#PEP-ucl}

   This section illustrates an example to configure a PEP  using
   the group-based ACL.

   The PEP which enforces group-based ACL may acquire group identities
   from the AAA server if working as a NAS authenticating both the
   source endpoint and the destination endpoint users. Another case for
   a PEP enforcing a group-based ACL is to obtain the group identity of
   the source endpoint directly from the packet field
   {{?I-D.smith-vxlan-group-policy}}.

   Assume the mapping between device group ID and IP addresses is
   predefined or acquired via device authentication. {{ex-PEP-ucl}}
   shows the ACL configuration delivered from the controller to the PEP. This
   example is consistent with the example presented in {{controller-ucl}}.

   The examples in this section do not intend to be exhaustive. In particular, explicit
   IP addresses ("destination-ipv4-network" or "destination-ipv6-network") are provided only for one single rule to illustrate
   how the mapping between a group ID and IP addresses is translated into an ACL rule entry.

~~~~ json
{::include-fold ./examples/valid-PEP-ucl.json}
~~~~
{: #ex-PEP-ucl title="Example of PEP Configuration"}

   {{ex-PEP-ucl-ipv6}} shows an example of the same policy but with a destination IPv6 prefix.

~~~~ json
{::include-fold ./examples/valid-PEP-ucl-ipv6.json}
~~~~
{: #ex-PEP-ucl-ipv6 title="Example of PEP Configuration (ipv6)"}

## Configuring PEPs Using Address-based ACLs {#PEP-acl}

   The section describes an example of configuring a PEP using
   IP address based ACL. IP address based access control policies could
   be applied to the PEP that may not understand the group information (e.g., firewall).

   Assume an employee in the R&D department accesses the network
   wirelessly from a non-corporate laptop.
   The SDN controller associates the user group to which the employee
   belongs with the user address according to steps 1 to 4 in {{overview}}.

   Assume the mapping between device group ID and IP addresses is
   predefined or acquired via device authentication. {{ex-PEP-acl}}
   shows an IPv4 address based ACL configuration delivered from
   the controller to the PEP. This example is consistent with the example
   presented in {{controller-ucl}}.

~~~~ json
{::include-fold ./examples/valid-PEP-acl.json}
~~~~
{: #ex-PEP-acl title="Example of PEP Configuration"}

{{ex-PEP-acl-ipv6}} shows an example of the same policy but with a destination IPv6 prefix.

~~~~ json
{::include-fold ./examples/valid-PEP-acl-ipv6.json}
~~~~
{: #ex-PEP-acl-ipv6 title="Example of PEP Configuration (IPv6)"}

# Acknowledgments
{:numbered="false"}

   This work has benefited from the discussions of User-group-based
   Security Policy over the years.  In particular, {{?I-D.you-i2nsf-user-group-based-policy}}
   and {{?I-D.yizhou-anima-ip-to-access-control-groups}} provided mechanisms to
   establish a mapping between the IP address/prefix of users and access
   control group IDs. The authors would like to thank Jianjie You, Myo Zarny, Christian Jacquenet, and Yizhou Li for their early contributions to these works.

   Thanks to Joe Clarke, Bill Fenner, Benoît Claise, Rob Wilton, David Somers-Harris,
   Alan Dekok, Heikki Vatiainen, Wen Xiang, Wei Wang, Hongwei Li, and Jensen Zhang for
   their review and comments.

   Thanks to Dhruv Dhody for the OPSDIR review, Alexander Pelov for INTDIR review, Valery Smyslov for the SECDIR review, and Acee Lindem for the YANGDOCTORS review.

   Thanks to Mahesh Jethanandani for the AD review.

   Thanks to Christopher Inacio, Andy Newton, Charles Eckel, Éric Vyncke, Deb Cooley, Gorry Fairhurst, Gunter Van de Velde, Jim Guichard, Ketan Talaulikar, and Mike Bishop for the IESG review.
