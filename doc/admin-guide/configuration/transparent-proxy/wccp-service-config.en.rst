.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.

.. include:: ../../../common.defs

.. _admin-wccp-service-config:

WCCP Service Configuration
**************************

The service definition file is used by :program:`traffic_wccp` and
:program:`traffic_server` directly.

The elements in the security definition file are inspired by the
`WCCP RFC (8/2012) <http://tools.ietf.org/html/draft-mclaggan-wccp-v2rev1-00>`_.
There is also an older version of the RFC that shows up commonly in search results,
`WCCP (4/2001) <https://tools.ietf.org/id/draft-wilson-wrec-wccp-v2-01.txt>`_,
and was the RFC reference used in the original WCCP support for |TS| several
years ago.

A sample service group file is included in the source tree under
:program:`traffic_wccp`.

Security Section
================

In the security section, you can define a password shared between the WCCP Client and the WCCP router.  This password is used to encrypt the WCCP traffic.  It is optional, but highly recommended.

Attributes in this section

*  option  - Must be set to MD5 if you want to encrypt your WCCP traffic

*  key – The same password that you set with the associated WCCP router.

Services Section
================

In the services section you can define a list of service groups.  Each top level entry is a separate service group.  

Service group attributes include

*  name – A name for the service.  Not used in the rest of the WCCP processing.

*  description – A description of the service.  Again, not used in the rest of the WCCP processing.

*  id - The security group ID.  It must match the service group ID that has been defined on the associated WCCP router.  This is the true service group identifier from the WCCP perspective.  

* type – This defines the type of service group either “STANDARD” or “DYNAMIC”.  There is one standard defined service group, HTTP with the id of 0.  The 4/2001 RFC indicates that id’s 0-50 are reserved for well known service groups.  But more recent 8/2012 RFC indicates that values 0 through 254 are valid service id’s for dynamic services.  To avoid differences with older WCCP routers, you probably want to  avoid dynamic service ID’s 0 through 50.  

* priority – This is a value from 0 to 255.  The higher number is a higher priority.  Well known (STANDARD) services are set to a value of 240.  If there are multiple service groups that could match a given packet, the higher priority service group is applied. RFC  For example, you have service group 100 defined for packets with destination port 80, and service group 101 defined for packets with source port 1024.  For a packet with destination port set to 80 and source port set to 1024, the priorities of the service groups would need to be compared to determine which service group applies.

* protocol – This is IP protocol number that should match.  Generally this is set to 6 (TCP) or 17 (UDP).

* assignment – WCCP supports multiple WCCP clients supporting a single service group.  However, the current WCCP client implementation in Traffic Server assumes there is only a single WCCP client, and so creates assignment tables that will direct all traffic to that WCCP client.  The assignment type is either hash or mask, and if it is not set, it defaults to hash.  If Traffic Server ever supports more than one cache, it will likely only support a balanced hash assignment.  The mask/value assignment seems to be better suited to situations where the traffic needs to be more strongly controlled.

* primary-hash – This is the element of the packet that is used to compute the primary key.  The value options are src_ip, dst_ip, src_port, or  dst_port. This entry is a list, so multiple values can be specified.  In that case, all the specified packet attributes will be used to compute the hash bucket.  In the current implementation, the primary hash value does not matter, since the client always generates a hash table that directs all matching traffic to it.  But if multiple clients are ever supported, knowledge of the local traffic distribution could be used to pick a packet attribute that will better spread traffic over the WCCP clients.
*  alt-hash – The protocol supports a two level hash.  This attribute is a list with the same value options as for primary-hash.  Again, since the current Traffic Server implementation only creates assignment tables to a single client, specifying the alt-hash values does nothing.  

* ports – This is a list of port values.  Up to 8 port values may be included in a service group definition.  

* port-type – This attribute can have the value of src or dst.  If not specified, it defaults to dst.  It indicates whether the port values should be interpreted as source ports or destination ports.

* forward – This is a list.  The list of the values can be GRE or L2.  This advertises how the client wants to process WCCP packets.  GRE means that the packets will be delivered in a GRE tunnel.  This is the default.  L2 means that the client is on the same network and can get traffic delivered to it from the router by L2 routing (MAC addresses).

* return – The WCCP protocol allows a WCCP client to decline a packet and return it back to the router.  The current WCCP client implementation never does this.  The value options are the same as for the forward attribute.

* routers – This is the list of router addresses the WCCP client communicates with.  The WCCP protocols allows for multiple WCCP routers to be involved in a service group.  The multiple router scenario has at most been lightly tested in the Traffic Server implementation.

* proc-name – This attribute is only used by traffic_wccp.  It is not used in the traffic_server WCCP support.  This is the path to a process’ PID file.  The service group is advertised to the WCCP router if the process identified in the PID file is currently operational.

