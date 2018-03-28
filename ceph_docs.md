########################
# Ceph Documentation
########################
1. Update modules and states on all Minions:

  salt '*' saltutil.sync_all

2. Run basic states on all Ceph nodes:

  salt "*" state.sls linux,openssh,salt,ntp,rsyslog

3. Generate admin and mon keyrings:

  salt -C 'I@ceph:mon:keyring:mon or I@ceph:common:keyring:admin' state.sls ceph.mon
  salt -C 'I@ceph:mon' saltutil.sync_grains
  salt -C 'I@ceph:mon:keyring:mon or I@ceph:common:keyring:admin' mine.update

4. Deploy Ceph mon nodes:

  a. If your Ceph version is older than Luminous:

      salt -C 'I@ceph:mon' state.sls ceph.mon

  b. If your Ceph version is Luminous or newer:

      salt -C 'I@ceph:mon' state.sls ceph.mon
      salt -C 'I@ceph:mgr' state.sls ceph.mgr

5. To prepare the Ceph CRUSH map model, uncomment the example pillar in the classes/cluster/<CLUSTER_NAME>/ceph/setup.yml file and modify it as required.

6. Deploy Ceph osd nodes:

  salt -C 'I@ceph:osd' state.sls ceph.osd
  salt -C 'I@ceph:osd' saltutil.sync_grains
  salt -C 'I@ceph:osd' state.sls ceph.osd.custom
  salt -C 'I@ceph:osd' saltutil.sync_grains
  salt -C 'I@ceph:osd' mine.update
  salt -C 'I@ceph:setup' state.sls ceph.setup

7. Deploy RADOS Gateway:

  salt -C 'I@ceph:radosgw' saltutil.sync_grains
  salt -C 'I@ceph:radosgw' state.sls ceph.radosgw

8. Set up the Keystone service and endpoints for Swift or S3:

  salt -C 'I@keystone:client' state.sls keystone.client

9. Connect Ceph to your MCP cluster:

  salt -C 'I@ceph:common and I@glance:server' state.sls ceph.common,ceph.setup.keyring,glance
  salt -C 'I@ceph:common and I@glance:server' service.restart glance-api
  salt -C 'I@ceph:common and I@glance:server' service.restart glance-glare
  salt -C 'I@ceph:common and I@glance:server' service.restart glance-registry
  salt -C 'I@ceph:common and I@cinder:controller' state.sls ceph.common,ceph.setup.keyring,cinder
  salt -C 'I@ceph:common and I@nova:compute' state.sls ceph.common,ceph.setup.keyring
  salt -C 'I@ceph:common and I@nova:compute' saltutil.sync_grains
  salt -C 'I@ceph:common and I@nova:compute' state.sls nova

10. Modify and apply the generated CRUSH map:

  a. View the CRUSH map generated in the /etc/ceph/crushmap file and modify it as required. Before applying the CRUSH map, verify that the settings are correct.

  b. Apply the following state:

      salt -C 'I@ceph:setup:crush' state.sls ceph.setup.crush

  c. Once the CRUSH map is set up correctly, add the following snippet to the classes/cluster/<CLUSTER_NAME>/ceph/osd.yml file to make the settings persist even after a Ceph OSD reboots:

ceph:
  osd:
    crush_update: false

  d. Apply the following state:

    salt -C 'I@ceph:osd' state.sls ceph.osd

Once done, if your Ceph version is Luminous or newer, you can access the Ceph dashboard through http://<active_mgr_node_IP>:7000/. Run ceph -s on a cmn node to obtain the active mgr node.
