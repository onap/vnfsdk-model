.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. SPDX-License-Identifier: CC-BY-4.0

VNF SDK sample Blueprints
=========================

pingpong.yaml
-------------
::

 # Copyright (c) 2016 GigaSpaces Technologies Ltd. All rights reserved
 #
 # Licensed under the Apache License, Version 2.0 (the "License");
 # you may not use this file except in compliance with the License.
 # You may obtain a copy of the License at
 #
 #        http://www.apache.org/licenses/LICENSE-2.0
 #
 # Unless required by applicable law or agreed to in writing, software
 # distributed under the License is distributed on an "AS IS" BASIS,
 #    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 #    * See the License for the specific language governing permissions and
 #    * limitations under the License.
 tosca_definitions_version: tosca_simple_profile_for_nfv_1_0

 description: >-
   Ping-Pong VNF example (draft 3).

   A simple VNF composed of two VDU templates, "ping" and "pong". For simplicity, we are
   not defining any scalability, so there will always be single insstances of each VDU.

 imports:
   - vnfsdk-profile-1.0.yaml # the types for our suggested extensions to the NFV profile

 topology_template:

   inputs:
     dpdk:
       description: >-
         Allows operators to override the dpdk requirement for the "ping" VDU.
       type: boolean
       default: true
     connectivity_indicator:
       description: >-
         Allows operators to override the connectivity indicator implementation for the
         "ping" VDU.
       default: scripts/vdu/ping_get_connectivity.sh # included in the CSAR

   substitution_mappings:
     # In a separate service chain blueprint, our entire blueprint here can be substituted as a
     # single complete VNF.
     node_type: tosca.nodes.nfv.VNF
     requirements:
       virtual_link: [ ingress_route, virtual_link ]

   node_templates:

     ping_vdu:
       type: tosca.nodes.nfv.VDU
       artifacts:
         image:
           type: tosca.artifacts.Deployment.Image.VM
           file: images/ping_vdu.qcow2 # included in the CSAR
       interfaces:
         Standard:
           configure: scripts/vdu/ping_configure.sh # included in the CSAR
       requirements:
         - dependency:
             node: ping_host
             relationship:
               type: vnfsdk.VirtualHostedOn
               properties:
                 indicators:
                   hits:
                     type: int
                     implementation: scripts/vdu/ping_get_hits.sh # included in the CSAR
                   connectivity:
                     type: string
                     implementation: { get_input: connectivity_indicator }
                 compute_dependencies:
                   profile:
                     value: high_performance
                     # "profile" is a special key that will pull in a complete bundles of
                     # dependencies, which can then override individually here. See
                     # compute-profiles.yaml, included in the CSAR, for the bundled values.
                   dpdk:
                     value: { get_input: dpdk }
                   sse:
                     value: version 2.0
                     optional: true
         - dependency: pong_vdu

     pong_vdu:
       type: tosca.nodes.nfv.VDU
       artifacts:
         image:
           type: tosca.artifacts.Deployment.Image.VM
           file: images/pong_vdu.qcow2 # included in the CSAR
       interfaces:
         Standard:
           configure: scripts/vdu/pong_configure.sh # included in the CSAR
       requirements:
         - dependency:
             node: pong_host
             relationship:
               type: vnfsdk.VirtualHostedOn
               properties:
                 compute_dependencies:
                   cpu_affinity:
                   value: 'true'
                 container:
                   value: 'false'

     ping_host:
       type: tosca.nodes.Compute

     pong_host:
       type: tosca.nodes.Compute

     ingress_vl:
       type: tosca.nodes.nfv.VL.ELine
       properties:
         cidr: 10.2.10.0/24
         gateway_ip: 10.2.10.1
         network_type: vlan
         physical_network: phynet2
         segmentation_id: '1001'
         vendor: TheCarrier

     ingress_route:
       type: tosca.nodes.nfv.CP
       description: >-
         Ingress connection point bound to the router VL and linked to the router VDU.
       properties:
         type: vPort
       requirements:
         - binding: ingress_vl
         - link: ping_vdu
         - virtual_binding: ingress_vl
         - virtual_link: ping_vdu

   policies:
     allocation:
       type: vnfsdk.Allocation
       targets:
         - ping_vdu
         - pong_vdu

     placement:
       type: vnfsdk.Placement
       targets:
         - ping_host
         - pong_host

vrouter.yaml
------------
::

  # Copyright (c) 2016 GigaSpaces Technologies Ltd. All rights reserved
  #
  # Licensed under the Apache License, Version 2.0 (the "License");
  # you may not use this file except in compliance with the License.
  # You may obtain a copy of the License at
  #
  #        http://www.apache.org/licenses/LICENSE-2.0
  #
  # Unless required by applicable law or agreed to in writing, software
  # distributed under the License is distributed on an "AS IS" BASIS,
  #    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  #    * See the License for the specific language governing permissions and
  #    * limitations under the License.

  tosca_definitions_version: tosca_simple_profile_for_nfv_1_0

  description: >-
    vRouter VNF example (draft 3).

    For the purposes of example, this VNF is composed of two VDU templates, one for routing that can
    scale, and a central VDU for storing the state that cannot scale. So, for example, in some
    configurations the VNF might have 3 routing VDU instances and a single data VDU instance (4 VDU
    instances total).

  imports:
    - vnfsdk-profile-1.0.yaml # the types for our suggested extensions to the NFV profile

  topology_template:

    substitution_mappings:
      # In a separate service chain blueprint, our entire blueprint here can be substituted as a
      # single complete VNF.
      node_type: tosca.nodes.nfv.VNF
      requirements:
        virtual_link: [ ingress_route, virtual_link ]

    node_templates:

      router_vdu:
        type: tosca.nodes.nfv.VDU
        description: >-
          In charge of handling the routing. Each instance of this VDU template can handle only a
          limited amount of bandwidth depending on host capabilities, but it can be scaled out within
          the VNF.
        artifacts:
          image:
            type: tosca.artifacts.Deployment.Image.VM
            file: images/router_vdu.qcow2 # included in the CSAR
        interfaces:
          Standard:
            configure: scripts/vdu/router_configure.sh # included in the CSAR
        requirements:
          - dependency:
              node: router_host
              relationship:
                type: vnfsdk.VirtualHostedOn
                properties:
                  indicators:
                    hits:
                      type: int
                      implementation: scripts/vdu/router_get_hits.sh # included in the CSAR
                    connectivity:
                      type: string
                      implementation: scripts/vdu/router_get_connectivity.sh # included in the CSAR
                  compute_dependencies:
                    dpdk:
                      value: 'true'
                      priority: 0.8 # a strong preference
                    sse:
                      value: version 2.0
                      priority: 0.2 # nice to have
                    container:
                      value: 'false'
                      # the default priority is 1.0, meaning an absolute requirement
          - dependency: data_vdu
        capabilities:
          monitoring_parameter: # note that this capability has an "ip_address" attribute
            properties:
              protocol: http
              port: 8081
              url_path: /data/telemetry

      data_vdu:
        type: tosca.nodes.nfv.VDU
        description: >-
          All the router VDUs connect to the same data VDU which stores the configuration and state of
          the VNF. We are assuming that this VNF product does not support scaling of the data VDU.
        artifacts:
          image:
            type: tosca.artifacts.Deployment.Image.VM
            file: images/data_vdu.qcow2 # included in the CSAR
        interfaces:
          Standard:
            configure: scripts/vdu/data_configure.sh # included in the CSAR
        requirements:
          - dependency:
              node: data_host
              relationship:
                type: vnfsdk.VirtualHostedOn
                properties:
                  compute_dependencies:
                    cpu_affinity:
                      value: 'true'
                    container:
                      value: 'false'
                      priority: 1.0 # hints that it's absolutely required
        capabilities:
          nfv_compute:
            # This is the "standard" way to define performance optimization in the NFV profile
            properties:
              mem_page_size: large
              cpu_allocation:
                core_count: 4
              numa_node_count: 2
              numa_nodes:
                primary:
                  id: 0
                  mem_size: 16 GB
                  vcpus:
                    a: 2
                    b: 3
                secondary:
                  id: 1
                  mem_size: 2 GB

      router_host:
        type: tosca.nodes.Compute
        capabilities:
          scalable:
            properties:
              max_instances: 4

      data_host:
        type: tosca.nodes.Compute
        requirements:
          - local_storage:
              node: database_storage
              relationship:
                properties:
                  location: /mnt/database

      database_storage:
        type: tosca.nodes.BlockStorage
        properties:
          size: 5 GB

      ingress_vl:
        type: tosca.nodes.nfv.VL.ELine
        properties:
          cidr: 10.2.10.0/24
          gateway_ip: 10.2.10.1
          network_type: vlan
          physical_network: phynet2
          segmentation_id: '1001'
          vendor: TheCarrier

      ingress_route:
        type: tosca.nodes.nfv.CP
        description: >-
          Ingress connection point bound to the router VL and linked to the router VDU.
        properties:
          type: vPort
        requirements:
          - binding: ingress_vl
          - link: router_vdu
          - virtual_binding: ingress_vl
          - virtual_link: router_vdu

    policies:
      allocation:
        type: vnfsdk.Allocation
        properties:
          prioritize: cost
        targets:
          - router_vdu
          - data_vdu

      placement:
        type: vnfsdk.Placement
        properties:
          prioritize: performance
        targets:
          - router_host
          - data_host
          - database_storage

      scaling:
        type: vnfsdk.Scaling
        properties:
          bandwidth_threshold: 1.5 GB
        targets:
          - router_vdu

      performance:
        type: vnfsdk.Assurance
        properties:
          responsivness: 0.1 Hz
        targets:
          - ingress_vl

vnfsdk-profile-1.0.yaml
-----------------------
::

  # Copyright (c) 2016 GigaSpaces Technologies Ltd. All rights reserved
  #
  # Licensed under the Apache License, Version 2.0 (the "License");
  # you may not use this file except in compliance with the License.
  # You may obtain a copy of the License at
  #
  #        http://www.apache.org/licenses/LICENSE-2.0
  #
  # Unless required by applicable law or agreed to in writing, software
  # distributed under the License is distributed on an "AS IS" BASIS,
  #    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  #    * See the License for the specific language governing permissions and
  #    * limitations under the License.

  # vnfsdk-profile-1.0 (draft 3)

  # Required extensions to the TOSCA Simple Profile for Network Functions Virtualization (NFV)
  # Version 1.0.

  relationship_types:

   vnfsdk.VirtualHostedOn:
       description: >-
        The standard VDU type in the simple NFV is completely separate from the "compute" world. The
        only relatiionship we have available to connect them is DependsOn. Thus, we must extend it in
        order to implement the connection.
      derived_from: tosca.relationships.DependsOn
      valid_target_types:
        - tosca.capabilities.Node
      properties:
        indicators:
          description: >-
            Performance, availability, and other indicators.
          type: map
          entry_schema: vnfsdk.Indicator
          required: false
        compute_dependencies:
          description: >-
            These dependencies will be used by a resource orchestrator/allocator to assign the best
            possible performance profile.
          type: map
          entry_schema: vnfsdk.ComputeDependency
          required: false

  policy_types:

    vnfsdk.Allocation:
      description: >-
        The orchestrator will apply an algorithm to make sure that VDUs will be allocated the best
        possible hosts according
      derived_from: tosca.policies.Root
      targets:
        - tosca.nodes.nfv.VDU
      properties:
        prioritize:
          description: >-
            Hint to allocation algorithm.
          type: vnfsdk.PolicyPriority
          default: balance

    vnfsdk.Placement:
      description: >-
        The orchestrator will apply an algorithm to make sure that our VIM resources will be allocated
        and placed near each other in the data center to guarantee best internal connectivity. This is
        necessary for the VNF to function as an efficient whole, despite being composed of many VIM
        resources.
      derived_from: tosca.policies.Placement
      targets:
        - tosca.nodes.Compute
        - tosca.nodes.BlockStorage
      properties:
        prioritize:
          description: >-
            Hint to placement algorithm.
          type: vnfsdk.PolicyPriority

    description: >-
        The orchestrator will monitor bandwidth and auto-scale (up and down) according to an algorithm
        configured by our defined parameters here.
      derived_from: tosca.policies.Scaling
      properties:
        bandwidth_threshold:
          description: >-
            Over this bandwidth the algorithm should start scaling up, and under it, it should scale
            down.
          type: scalar-unit.size
        polling_interval:
          description: >-
            How often to check bandwidth.
          type: scalar-unit.time
          default: 10 s
      targets:
        - tosca.nodes.nfv.VDU

    vnfsdk.Assurance:
      description: >-
        The orchestrator (SDN-O?) will make sure the virtual link is configured to perform within our
        defined performance envelope here.
      derived_from: tosca.policies.Performance
      properties:
        responsivness:
          description: >-
            How often assurance should be checked.
          type: scalar-unit.frequency
      targets:
        - tosca.nodes.nfv.VL.ELine

  data_types:

    vnfsdk.ComputeDependency:
      derived_from: tosca.datatypes.Root
      properties:
        value:
          description: >-
            The dependency value.
          type: string
        optional:
          description: >-
            Whether the dependency is optional. When set to true implies priority 0.5.
          type: boolean
          default: false
        priority:
          description: >-
            Allows fine-tuning of the dependency's priority.
          type: float
          constraints:
            - in_range: [ 0.0, 1.0 ]
          default: 1.0

    vnfsdk.Indicator:
      derived_from: tosca.datatypes.Root
      properties:
        type:
          description: >-
            The type of the indicator.
          type: string
        implementation:
          description: >-
            The indicator's implementation.
          type: string

    vnfsdk.PolicyPriority:
      derived_from: string
      constraints:
        - valid_values: [ performance, cost, balance ]


compute-profiles.yaml
---------------------
::

  # Copyright (c) 2016 GigaSpaces Technologies Ltd. All rights reserved
    #
  # Licensed under the Apache License, Version 2.0 (the "License");
  # you may not use this file except in compliance with the License.
  # You may obtain a copy of the License at
  #
  #        http://www.apache.org/licenses/LICENSE-2.0
  #
  # Unless required by applicable law or agreed to in writing, software
  # distributed under the License is distributed on an "AS IS" BASIS,
  #    * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  #    * See the License for the specific language governing permissions and
  #    * limitations under the License.

  performance_profiles:

    tosca.nodes.nfv.VDU:

      high_availability:
        cpu_allocation:
          cpu_affinity: dedicated
          thread_allocation: isolate
          socket_count: 2
          core_count: 2
          thread_count: 4
        dpdk: true
        cpu_affinity:
          default: false
          value: { get_input: cpu_affinity }

      low_availability:
        cpu_allocation:
          thread_allocation: isolate
          socket_count: 1
          core_count: 1
          thread_count:
            default: 4
            value: { get_input: thread_count }
            0000c0onstraint: { in_range: [ 1, 4 ] }
        dpdk: false
        cpu_affinity: true

Code for this document is available under the Apache License, Version 2.0
