###################################
Create Networks On KVM Node
##################################

1. mcp-pxe : 10.100.0.0/24
2. mcp-control : 10.101.0.0/24
3. mcp-data(tenant) :
4. mcp-public : 172.17.18.32/27
5. mcp-proxy : 172.17.18.0/27

infra01 : 172.17.18.36
kvm01 :
kvm02 :
kvm03 :

snv.mirantis.local

######################
Instalation Steps
######################

1. Deploy the Foundation physical node.

2. Configure bridges on the Foundation node:

    br-mgm for the management network
    br-ctl for the control network

3. Log in to the Foundation node.

    mkdir -p /var/lib/libvirt/images/cfg01/
    wget http://images.mirantis.com/cfg01-day01.qcow2 -O /var/lib/libvirt/images/cfg01/system.qcow2
    cp /path/to/prepared-drive/cfg01-config.iso /var/lib/libvirt/images/cfg01/cfg01-config.iso

4. Create the Salt Master VM domain definition:

    virt-install --name cfg01.mirantis.local \
    --disk path=/var/lib/libvirt/images/cfg01/system.qcow2,bus=virtio,cache=none \
    --disk path=/var/lib/libvirt/images/cfg01/cfg01-config.iso,device=cdrom \
    --network bridge:br-mgm,model=virtio \
    --network bridge:br-ctl,model=virtio \
    --ram 16384 --vcpus=8 --accelerate \
    --boot hd --vnc --noreboot --autostart

5. Start the Salt Master node VM:

    virsh start cfg01.mirantis.local
    virsh console cfg01.mirantis.local

    Note: all class will be placed at /srv/salt


6. Verify that the following states are successfully applied during the execution of cloud-init:

    salt-call state.sls linux.system,linux,openssh,salt
    salt-call state.sls maas.cluster,maas.region,reclass

7. In case of using kvm01 as the Foundation node, perform the following steps on it:

    a. Depending on the deployment type, proceed with one of the options below:


    a. deb [arch=amd64] http://repo.saltstack.com/apt/ubuntu/16.04/amd64/2016.3/xenial main
    b. Install the salt-minion package.

      apt-get install salt-minion=2016.3.8

       Note : fix key
       sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys <Key>


    c. Modify /etc/salt/minion.d/minion.conf:

        id: <kvm01_FQDN>
        master: <Salt_Master_IP_or_FQDN>
    d. Restart the salt-minion service

    e. Check the output of salt-key command on the Salt Master node to verify that the minion ID of kvm01 is present.


##################
Important TODO
#################

1. edit sshd_config for root Login

2. Add user ssh key

3. Copy qcow2 images to cfg nodes

    -> Login to cfg01
      a. mkdir /srv/salt/env/prd/images

      b. cd /srv/salt/env/prd/images

      c. wget http://images.mirantis.com.s3.amazonaws.com/ubuntu-16-04-x64-mcp2018.1.qcow2

        Images from images.mirantis.net

      d. edit infra/init.yml

        Replace :

        salt_control_trusty_image: http://images.mirantis.com/ubuntu-14-04-x64-mcp${_param:apt_mk_version}.qcow2

        With :

        salt_control_trusty_image: salt://images/ubuntu-14-04-x64-mcp${_param:apt_mk_version}.qcow2

4. add DHCP interfaces to all virtual networks

    Adding ens2 interface for deploynetwork

      1. edit stacklight/networking/virtual.yml

        add below line to interface

            ens2: ${_param:linux_dhcp_interface}

              ./stacklight/networking/virtual.yml
              ./openstack/networking/virtual.yml
              ./opencontrail/networking/virtual.yml
              ./cicd/networking/virtual.yml

6. proxy commadn to connect maas

  ssh -f root@hp01 -L 8080:10.100.0.15:8080 -N

7. Enable Swap on cfg node if it is less memory node.

      sudo fallocate -l 1G /swapfile
      sudo chmod 600 /swapfile
      sudo mkswap /swapfile
      sudo swapon /swapfile
      echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

8. Create manage-project in cicd01 Nodes

    chmod +x /usr/local/bin/manage-projects


9. Fix Gerrit tags issues on CICD Nodes

    Workaround:
      a. In Gerrit UI add access rule `Forge Committer Identity` for group `Administrators` to `refs/tags/*` for project `All-Projects`
      b. In Gerrit UI remove projects `mcp-ci/pipeline-library` and ` mk/mk-pipelines`
      c. On cid01 node remove `/srv/jeepyb` directory
      d. On cid01 node run Salt state `gerrit.client`

10. Verify that the volume is mounted on Docker Swarm nodes:

      salt '*' cmd.run 'systemctl -a|grep "GlusterFS File System"|grep -v mounted'

11. check all repos are in same tag and also for duplicate repos


###########
MaaS node
###########

1. Log in to the MAAS web UI through salt_master_management_address/MAAS with the following credentials:

  Username: mirantis
  Password: r00tme
2. Go to the Subnets tab.

3. Select the fabric that is under the deploy network.

4. In the VLANs on this fabric area, click the VLAN under the VLAN column where the deploy network subnet is.

5. In the Take action drop-down menu, select Provide DHCP.

6. Adjust the IP range as required.

    Note
    The number of IP addresses should not be less than the number of the planned VCP nodes.

7. Click Provide DHCP to submit.


########################
Provision KVM Nodes
########################

1. Define all physical nodes under classes/cluster/<cluster>/infra/maas.yml using the following structure.

  For example, to define the kvm02 node:

maas:
  region:
    machines:
      kvm02:
        interface:
          mac: 00:25:90:eb:92:4a
        power_parameters:
          power_address: kvm02.ipmi.net
          power_password: password
          power_type: ipmi
          power_user: ipmi_user
Note


2. To get MAC addresses from IPMI, you can use the ipmi tool. Usage example for Supermicro:

  ipmitool -U ipmi_user-P passowrd -H kvm02.ipmi.net raw 0x30 0x21 1| tail -c 18

3. Once you have defined all physical servers in your Reclass model, enforce the nodes:

  salt-call maas.process_machines

4. All nodes are automatically commissioned.

  Verify the status of servers either through the MAAS web UI or using the salt call command:

  salt-call maas.machines_status

5. The successfully commissioned servers appear in the ready status.

  (Optional) Enforce the interfaces configuration defined in the model for servers:

  salt-call state.sls maas.machines.assing_ip

6. (Optional) Enforce the disk custom configuration defined in the model for servers:

  salt-call state.sls maas.machines.storage

  Verify that all servers have correct NIC names and configurations.

7. Log in to the MAAS node console.

  Type the salt-call command:

  salt-call maas.deploy_machines

8. Check the status of the nodes:

9. When all servers have been provisioned, perform the verification of the servers automatic registration by running the

  salt-key



#############
Add New User
#############

1. Copy any example file from

  cfg01:/srv/salt/reclass/classes/system/openssh/server/team/
  to
  cfg01:/srv/salt/reclass/classes/cluster/snv/infra/chandra.yml

  edit the file as required

2. add the user class to init files

  vim /srv/salt/reclass/classes/cluster/snv/infra/chandra.yml

  - cluster.snv.infra.chandra

3. run salt-call to update user

  salt '*' cmd.run 'salt-call state.sls linux.system.user,openssh'

4. If encounter with any error try to run

  reclass-salt --top

##########################
Deploy physical servers
##########################

1. Verify that the cfg01 key has been added to Salt and your host FQDN is shown properly in the Accepted Keys field in the output of the following command:

  salt-key

2. Verify that all pillars and Salt data are refreshed:

  salt "*" saltutil.refresh_pillar
  salt "*" saltutil.sync_all

3. Verify that the Reclass model is configured correctly. The following command output should show top states for all nodes:

  reclass-salt --top

4. To verify that the rebooting of the nodes, which will be performed further, is successful, create the trigger file:

  salt -C 'I@salt:control or I@nova:compute or I@neutron:gateway' cmd.run "touch /run/is_rebooted"

5. For KVM nodes:

  salt --async -C 'I@salt:control' cmd.run 'salt-call state.sls linux.system.user,openssh,linux.network;reboot'

6. For compute nodes:

  salt --async -C 'I@nova:compute' cmd.run 'salt-call state.sls linux.system.user,openssh,linux.network;reboot'

7. For gateway nodes, execute the following command only for the deployments with OVS setup with physical gateway nodes:

  salt --async -C 'I@neutron:gateway' cmd.run 'salt-call state.sls  linux.system.user,openssh,linux.network;reboot'


8. Verify that the targeted nodes are up and running:

  salt -C 'I@salt:control or I@nova:compute or I@neutron:gateway' test.ping

9. Check the previously created trigger file to verify that the targeted nodes are actually rebooted:

  salt -C 'I@salt:control or I@nova:compute or I@neutron:gateway' cmd.run 'if [ -f "/run/is_rebooted" ];then echo "Has not been rebooted!";else echo "Rebooted";fi'

  All nodes should be in the Rebooted state.

10. Verify that the hardware nodes have the required network configuration. For example, verify the output of the ip a command:

  salt -C 'I@salt:control or I@nova:compute or I@neutron:gateway'  cmd.run "ip a"

################
Deply VCP
###############

1. On the Salt Master node, prepare the node operating system by running the Salt linux state:

  salt-call state.sls linux -l info

2. Verify that the Salt Minion nodes are synchronized by running the following command on the Salt Master node:

  salt '*' saltutil.sync_all

3. Perform the initial Salt configuration:

  salt 'kvm*' state.sls salt.minion

4. Set up the network interfaces and the SSH access:

  salt -C 'I@salt:control' cmd.run 'salt-call state.sls linux.system.user,openssh,linux.network;reboot'

5. Run the libvirt state:

  salt 'kvm*' state.sls libvirt

6. ( optional only needed when ovs is enabled ) Add system.salt.control.cluster.openstack_gateway_single to infra/kvm.yml to enable a gateway VM for your OpenStack environment.

7. Run salt.control to create virtual machines. This command also inserts minion.conf files from KVM hosts:

  salt 'kvm*' state.sls salt.control

8. Verify nodes

  salt-key


#######################
Deploy DriveTrain
######################

1. To set up the physical nodes for CI/CD:

Enable virtual IP:

  salt -C 'I@salt:control' state.sls keepalived

Deploy the GlusterFS cluster:

  salt -C 'I@glusterfs:server' state.sls glusterfs.server.service
  salt -C 'I@glusterfs:server and *01*' state.sls glusterfs.server.setup

Note: If using single kvm machine. then apply this fix

  ```
  On kvm01 node:

    vim vim /usr/lib/python2.7/dist-packages/salt/modules/glusterfs.py

      Comment out this lines

      #    if replica:
      #        cmd += 'replica {0} '.format(replica)
  ```

2. CD/CD deployment

  a. Perform the initial configuration:

    salt 'ci*' cmd.run 'salt-call state.sls salt.minion'
    salt 'ci*' state.sls salt.minion,linux,openssh,ntp

  b. Mount Gluster volumes from the KVM nodes:

    salt -C 'I@glusterfs:client and I@docker:host' state.sls glusterfs.client

  c. Configure virtual IP and HAProxy balancing:

    salt -C 'I@haproxy:proxy and I@docker:host' state.sls haproxy,keepalived

  d. Install Docker:

    salt -C 'I@docker:host' state.sls docker.host

  e. Initial Docker swarm leader:

    salt -C 'I@docker:swarm:role:master' state.sls docker.swarm

  f. Update the Salt mine to enable other swarm nodes to connect to leader:

    salt -C 'I@docker:swarm' state.sls salt
    salt -C 'I@docker:swarm' mine.flush
    salt -C 'I@docker:swarm' mine.update

  g. Synchronize modules and states:

    salt -C 'I@docker:swarm' saltutil.sync_all

  h. Complete the Docker swarm deployment:

    salt -C 'I@docker:swarm' state.sls docker.swarm

  i. Verify that all nodes are in the cluster:

    salt -C 'I@docker:swarm:role:master' cmd.run 'docker node ls'

  j. Apply the aptly.publisher state:

    salt -C 'I@aptly:publisher' state.sls aptly.publisher

  k. Start the CI/CD containers, for example, MySQL, Aptly, Jenkins, Gerrit, and others:

    salt -C 'I@docker:swarm:role:master' state.sls docker.client

  l. (optional) Configure the Aptly service:

    salt -C 'I@aptly:server' state.sls aptly

  m. Configure the OpenLDAP service for Jenkins and Gerrit:

    salt -C 'I@openldap:client' state.sls openldap

  n. Configure the Gerrit service, create users, projects, and so on:

    salt -C 'I@gerrit:client' state.sls gerrit

    Note:

    Apply this bug fix before you run gerrit state

    -> login to cicd01 node

        i. root@dvtcoscid01:~# vim /usr/local/bin/manage-projects
          #!/usr/bin/python
          # PBR Generated from u'console_scripts'
          import sys
          from jeepyb.cmd.manage_projects import main
          if __name__ == "__main__":
              sys.exit(main())

        ii. chmod +x /usr/local/bin/manage-projects

    o. Configure the Jenkins service, create users, add pipelines, and so on:

        salt -C 'I@jenkins:client' state.sls jenkins



###########
Verfication
############

Log in to the Jenkins web UI as admin.

The password for the admin user is defined in the classes/cluster/<cluster_name>/cicd/control/init.yml
file of the Reclass model under the openldap_admin_password parameter variable.

In the global view, verify that the git-mirror-downstream-mk-pipelines and git-mirror-downstream-pipeline-library
pipelines have successfully mirrored all content.



#########################
Deploy Openstack
#######################

############
Pre-Steps
############

a. Set up network interfaces and the SSH access on all compute nodes:

    salt -C 'I@nova:compute' cmd.run 'salt-call state.sls \
       linux.system.user,openssh,linux.network;reboot'

b. If you run OVS, run the same command on physical gateway nodes as well:

    salt -C 'I@neutron:gateway' cmd.run 'salt-call state.sls \
       linux.system.user,openssh,linux.network;reboot'

c. Verify that all nodes are ready for deployment:

    salt '*' state.sls linux,ntp,openssh,salt.minion

#################
OpenStack Infra
#################

1. To deploy Keepalived:

    salt -C 'I@keepalived:cluster' state.sls keepalived -b 1

2. Determine the VIP address for the current environment:

    salt -C 'I@keepalived:cluster' pillar.get keepalived:cluster:instance:VIP:address

3. Verify if the obtained VIP address is assigned to any network interface on one of the controller nodes:

    salt -C 'I@keepalived:cluster' cmd.run "ip a | grep <ENV_VIP_ADDRESS>"

4. To deploy NTP:

    salt '*' state.sls ntp

5. To deploy GlusterFS:

    salt -C 'I@glusterfs:server' state.sls glusterfs.server.service
    salt -C 'I@glusterfs:server' state.sls glusterfs.server.setup -b 1

    To verify GlusterFS:
    salt -C 'I@glusterfs:server' cmd.run "gluster peer status; gluster volume status" -b 1

6. Apply the rabbitmq state:

    salt -C 'I@rabbitmq:server' state.sls rabbitmq
    Verify the RabbitMQ status:

    salt -C 'I@rabbitmq:server' cmd.run "rabbitmqctl cluster_status"

7. Apply the galera state:

    salt -C 'I@galera:master' state.sls galera
    salt -C 'I@galera:slave' state.sls galera

    Verify that Galera is up and running:

    salt -C 'I@galera:master' mysql.status | grep -A1 wsrep_cluster_size
    salt -C 'I@galera:slave' mysql.status | grep -A1 wsrep_cluster_size

8. To deploy HAProxy:

    salt -C 'I@haproxy:proxy' state.sls haproxy
    salt -C 'I@haproxy:proxy' service.status haproxy
    salt -I 'haproxy:proxy' service.restart rsyslog

9. To deploy Memcached:

    salt -C 'I@memcached:server' state.sls memcached

10. To deploy Keystone:

    Set up the Keystone service:

    salt -C 'I@keystone:server' state.sls keystone.server -b 1

    Restart Apache2

    salt -C 'I@keystone:server' service.restart apache2

    Populate keystone services/tenants/admins:

    salt -C 'I@keystone:client' state.sls keystone.client
    salt -C 'I@keystone:server' cmd.run ". /root/keystonerc; openstack service list"

    Note:
11. To deploy Glance:

  Install Glance and verify that GlusterFS clusters exist:

    salt -C 'I@glance:server' state.sls glance -b 1
    salt -C 'I@glusterfs:client' state.sls glusterfs.client

  Update Fernet tokens before doing request on the Keystone server. Otherwise, you will get the following error: No encryption keys found; run keystone-manage fernet_setup to bootstrap one:

    salt -C 'I@keystone:server' state.sls keystone.server
    salt -C 'I@keystone:server' cmd.run ". /root/keystonerc; glance image-list"

12. To deploy the Nova:

  Install Nova:

    salt -C 'I@nova:controller' state.sls nova -b 1
    salt -C 'I@keystone:server' cmd.run ". /root/keystonerc; nova service-list"

    On one of the controller nodes, verify that the Nova services are enabled and running:

    root@cfg01:~# ssh ctl01 "source keystonerc; nova service-list"

13. To deploy Cinder:

  Install Cinder:

    salt -C 'I@cinder:controller' state.sls cinder -b 1

  On one of the controller nodes, verify that the Cinder service is enabled and running:

    salt -C 'I@keystone:server' cmd.run ". /root/keystonerc; cinder list"

14. To install Neutron:

    salt -C 'I@neutron:server' state.sls neutron -b 1
    salt -C 'I@neutron:gateway' state.sls neutron
    salt -C 'I@keystone:server' cmd.run ". /root/keystonerc; neutron agent-list"

15. To install Horizon:

    salt -C 'I@horizon:server' state.sls horizon
    salt -C 'I@nginx:server' state.sls nginx

#####################
Deploy Proxy Nodes
#####################

To install proxy nodes:

1. Add NAT for br2:

root@kvm01:~# iptables -t nat -A POSTROUTING -o br2 -j MASQUERADE
root@kvm01:~# echo “1” > /proc/sys/net/ipv4/ip_forward
root@kvm01:~# iptables-save > /etc/iptables/rules.v4

2. Deploy linux, openssh, and salt states to the proxy nodes:

root@cfg01:~# salt 'prx*' state.sls linux,openssh,salt

3. Verify the connection to Horizon:

You may need first to configure a SOCKS proxy or similar to the environment network to gain access from within your browser.
In a browser, connect to each of the proxy IPs and its VIP to verify that they are active.


####################
Deploy Compute Node
####################

1. Verify that the new machines have connectivity with the Salt Master node:

    salt 'cmp*' test.ping

2.  Run the reclass.storage state to refresh the deployed pillar data:

    salt 'cfg*' state.sls reclass.storage

3.  Apply the Salt data sync and base states for Linux, NTP, OpenSSH, and Salt on the target nodes:

    salt 'cmp*' saltutil.sync_all
    salt 'cmp*' saltutil.refresh_pillar
    salt 'cmp*' state.sls linux,ntp,openssh,salt

4. Apply all states for the target nodes:

    salt 'cmp*' state.apply

5.

##############
ssh -f root@mcplab -L 8081:192.168.100.15:8081 -N


jogsr0lZwjXdUCAx
