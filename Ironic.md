## Ironic
-------------------------

### Modify the deployment model
--------------------

1. #. In the top Reclass ``cluster/<cluster_name>/infra/init.yml`` file, add:

   .. code-block:: yaml
   
       parameters:
        _param:
          openstack_baremetal_node01_address: 172.16.10.110
          openstack_baremetal_address: 192.168.90.10
          openstack_baremetal_node01_baremetal_address: 192.168.90.11
          openstack_baremetal_neutron_subnet_cidr: 192.168.90.0/24
          openstack_baremetal_neutron_subnet_allocation_start: 192.168.90.100
          openstack_baremetal_neutron_subnet_allocation_end: 192.168.90.150
          openstack_baremetal_node01_hostname: bmt01

   .. note::

      The ``openstack_baremetal_neutron_subnet_`` parameters must match your
      baremetal network settings. The baremetal nodes must connected to the
      network before the deployment. During the deployment, MCP automatically
      registers this network in the OpenStack Networking service.

#. Modify the ``cluster/<cluster_name>/infra/config.yml``:


   .. code-block:: yaml

      classes:
      - system.salt.master.formula.pkg.baremetal
      - system.reclass.storage.system.openstack_baremetal_single

      parameters:
        reclass:
          storage:
            class_mapping:
              expression: <<node_hostname>>__startswith__bmt
              node_class:
                value_template:
                  - cluster.<<node_cluster>>.openstack.baremetal
              cluster_param:
                openstack_baremetal_node01_address:
                  value_template: <<node_control_ip>>
            node:
              openstack_baremetal_node01:
                params:¬
                  single_baremetal_address: ${_param:openstack_baremetal_node01_baremetal_address}
                  keepalived_openstack_baremetal_vip_priority: 100
                  ironic_api_type: 'deploy'
                  tenant_address: 10.1.0.110
                  external_address: 10.16.0.110

#. Modify the file to add custom network interfaces ``cluster/<cluster_name>/infra/kvm.yml``:


   .. code-block:: yaml

      classes:
      - cluster.<cluster_name>.infra.baremetal
      parameters:
        virt:
          nic:
            bmtnet:
              eth2:
                bridge: br_mgm
              eth1:
                bridge: br_ctl
              eth0:
                bridge: br_ironic

#. Modify the file at ``cluster/<cluster_name>/infra/baremetal.yml``:


   .. code-block:: yaml

      parameters:
        salt:
          control:
            size:
              bmt:
                cpu: 4
                ram: 16384
                disk_profile: small
                net_profile: bmtnet
            cluster:
              internal:
                domain: ${_param:cluster_domain}
                engine: virt
                node:
                  bmt01:
                    name: ${_param:openstack_baremetal_node01_hostname}
                    provider: ${_param:infra_kvm_node03_hostname}.${_param:cluster_domain}
                    image: ${_param:salt_control_xenial_image}
                    size: bmt

#. Modify the OpenStack nodes:

   * ``cluster/<cluster_name>/openstack/init.yml``:

     .. code-block:: yaml

         parameters:
          _param:
            ironic_version: ${_param:openstack_version}
            ironic_api_type: 'public'
            ironic_service_host: ${_param:cluster_vip_address}
            cluster_baremetal_local_address: ${_param:cluster_local_address}
            mysql_ironic_password: workshop
            keystone_ironic_password: workshop
          linux:
            network:
              host:
                bmt01:
                  address: ${_param:openstack_baremetal_node01_address}
                  names:
                  - bmt01
                  - bmt01.${_param:cluster_domain}

   * ``cluster/<cluster_name>/openstack/control.yml``:

     .. code-block:: yaml

         classes:
         - system.haproxy.proxy.listen.openstack.ironic
         - system.galera.server.database.ironic
         - service.ironic.client
         - system.ironic.api.cluster

  * ``cluster/<cluster_name>/openstack/database.yml``:

     .. code-block:: yaml

        classes:
        - system.galera.server.database.ironic

   * ``cluster/<cluster_name>/openstack/control_init.yml``:

     .. code-block:: yaml

         classes:
         - system.keystone.client.service.ironic

   * ``cluster/<cluster_name>/openstack/gateway.yml``:

     .. code-block:: yaml

         classes:
         - system.neutron.gateway.ironic

   * ``cluster/<cluster_name>/openstack/baremetal.yml``:

     .. code-block:: yaml

         classes:
         - system.linux.system.repo.mcp.openstack
         - system.linux.system.repo.mcp.extra
         - system.linux.system.repo.saltstack.xenial
         - system.ironic.api.cluster # deploy only api (heartbeat and lookup endpoints are open)
         - system.ironic.conductor.cluster
         - system.ironic.tftpd_hpa
         - system.nova.compute_ironic.cluster
         - system.apache.server.single
         - system.apache.server.site.ironic
         - system.keystone.client.core
         - system.neutron.client.service.ironic
         - cluster.<cluster_name>.infra
         parameters:
           _param:
             primary_interface: ens4
             baremetal_interface: ens5
             linux_system_codename: xenial
             interface_mtu: 1450
             cluster_vip_address: ${_param:openstack_control_address}
             cluster_baremetal_vip_address: ${_param:single_baremetal_address}
             cluster_baremetal_local_address: ${_param:single_baremetal_address}
             linux_system_codename: xenial
           linux:
             network:
               concat_iface_files:
               - src: '/etc/network/interfaces.d/50-cloud-init.cfg'
                 dst: '/etc/network/interfaces'
               bridge: openvswitch
               interface:
                 dhcp_int:
                   enabled: true
                   name: ens3
                   proto: dhcp
                   type: eth
                   mtu: ${_param:interface_mtu}
                 primary_interface:
                   enabled: true
                   name: ${_param:primary_interface}
                   proto: static
                   address: ${_param:single_address}
                   netmask: 255.255.255.0
                   mtu: ${_param:interface_mtu}
                   type: eth
                 baremetal_interface:
                   enabled: true
                   name: ${_param:baremetal_interface}
                   mtu: ${_param:interface_mtu}
                   proto: static
                   address: ${_param:cluster_baremetal_local_address}
                   netmask: 255.255.255.0
                   type: eth
                   mtu: ${_param:interface_mtu}

   * add br-baremetal bridge in gateway_networking

      .. code-block:: yaml

      # Ironic
         ens7:
           enabled: true
           proto: manual
           type: eth
           ovs_bridge: br-baremetal
           ovs_type: OVSPort
         br-baremetal:
           enabled: true
           type: ovs_bridge

   * update kvm04.yml to reflect gtw node with 4 nic

     .. code-block:: yaml

    virt:
       nic:
         gtwnet:
           - name: eth2
             bridge: br_private
             model: virtio
           - name: eth3
             bridge: br_proxy
             model: virtio

  * Run linux.netowkr state to create new interface and bridge

         salt 'gtw*' state.sls linux.network



### Apply salt sates
--------------------

1. Setup BMT node

        setup vm in infra/kvm04.yml and in config

1. Add the configured Octavia roles to the corresponding nodes:

        salt-call state.sls reclass.storage

    Refresh pillars:

        salt '*' saltutil.refresh_pillar

2. Update the Salt Minion configuration:

        salt -C 'I@ironic:api' state.sls salt.minion.service

3. Create the Octavia database:

        salt -C 'I@galera:master' state.sls galera
        salt -C 'I@galera:slave' state.sls galera -b 1

4. Configure HAProxy for Octavia API:

        salt -C 'I@haproxy:proxy' state.sls haproxy

5. Create an Octavia user and endpoints in Keystone:

        salt -C 'I@keystone:client' state.sls keystone.client

6. Install Ironic API:

        salt -C 'I@ironic:api' state.sls ironic.api -b 1

7 Install Ironic Conductor:

        salt -C 'I@ironic:conductor' state.sls ironic.conductor

8. Install Ironic Client:

        salt -C 'I@ironic:client' state.sls ironic.client

9. Install software required by Ironic, such as Apache and TFTP server:

        salt -C 'I@ironic:conductor' state.sls apache
        salt -C 'I@tftpd_hpa:server' state.sls tftpd_hpa

10. Install nova-compute with ironic virt-driver:

        salt -C 'I@nova:compute' state.sls nova.compute
        salt -C 'I@nova:compute' cmd.run 'systemctl restart nova-compute'

11. Append physnet bridge mapping and create baremetal subnet for Ironic:

           salt -C 'I@neutron:gateway' state.sls neutron
           salt -C 'I@neutron:client' state.sls neutron.client

11. Log in to an OpenStack Controller node.

    Verify that the Ironic services are enabled and running:

        salt -C 'I@ironic:client' cmd.run '. /root/keystonercv3; ironic driver-list'
