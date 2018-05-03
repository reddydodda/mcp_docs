## Ironic
-------------------------

### Modify the deployment model
--------------------

1. In the Reclass cluster model under  cluster/<cluster>/infra/init.yml file, add:

        parameters:
          _param:
             openstack_baremetal_node01_address: 172.16.10.110
             openstack_baremetal_address: 192.168.90.10
             openstack_baremetal_node01_baremetal_address: 192.168.90.11
             openstack_baremetal_neutron_subnet_cidr: 192.168.90.0/24
             openstack_baremetal_neutron_subnet_allocation_start: 192.168.90.100
             openstack_baremetal_neutron_subnet_allocation_end: 192.168.90.150
             openstack_baremetal_node01_hostname: bmt01

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

11. Log in to an OpenStack Controller node.

    Verify that the Ironic services are enabled and running:

        salt -C 'I@ironic:client' cmd.run '. /root/keystonercv3; ironic driver-list'
