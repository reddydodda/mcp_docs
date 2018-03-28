###############
# Deploy OSS
###############

1 Refresh Salt pillars and synchronize Salt modules on all Salt Minion nodes:

  salt '*' saltutil.refresh_pillar
  salt '*' saltutil.sync_all

2. Set up GlusterFS:

  salt -b 1 -C 'I@glusterfs:server' state.sls glusterfs.server

3. Mount the GlusterFS volume on Docker Swarm nodes:

  salt -C 'I@glusterfs:client' state.sls glusterfs.client

4. Verify that the volume is mounted on Docker Swarm nodes:

  salt '*' cmd.run 'systemctl -a|grep "GlusterFS File System"|grep -v mounted'

5. Configure HAProxy and Keepalived for the load balancing of incoming traffic:

  salt -C "I@haproxy:proxy" state.sls haproxy,keepalived

6. Set up Docker Swarm:

  salt -C 'I@docker:host' state.sls docker.host
  salt -C 'I@docker:swarm:role:master' state.sls docker.swarm
  salt -C 'I@docker:swarm:role:master' state.sls salt
  salt -C 'I@docker:swarm:role:master' mine.flush
  salt -C 'I@docker:swarm:role:master' mine.update
  salt -C 'I@docker:swarm' state.sls docker.swarm
  salt -C 'I@docker:swarm:role:master' cmd.run 'docker node ls'

7. Configure the OSS services:

  salt -C 'I@devops_portal:config' state.sls devops_portal.config
  salt -C 'I@rundeck:server' state.sls rundeck.server


#########################
stacklight
#########################

1. Deploy Docker Swarm master:

salt -C 'I@docker:host' state.sls docker.host
salt -C 'I@docker:swarm:role:master' state.sls docker.swarm

2. Deploy Docker Swarm workers:

salt -C 'I@docker:swarm:role:manager' state.sls  docker.swarm -b 1

3. Deploy Keepalived:

salt -C 'I@keepalived:cluster' state.sls keepalived -b 1

4. Deploy NGINX proxy:

salt -C 'I@nginx:server' state.sls nginx



#### Elasticsearch and Kibana

1. Deploy the Elasticsearch and Kibana services:

salt -C 'I@elasticsearch:server' state.sls elasticsearch.server -b 1
salt -C 'I@kibana:server' state.sls kibana.server -b 1

2. Deploy the Elasticsearch and Kibana clients that will configure the corresponding servers:

salt -C 'I@elasticsearch:client' state.sls elasticsearch.client
salt -C 'I@kibana:client' state.sls kibana.client


3. To verify the Elasticsearch cluster:

Log in to one of the log hosts.

Run the following command:

curl http://log:9200


4. To verify the Kibana dashboard:

Log in to the Salt Master node.

Identify the prx VIP of your MCP cluster:

  salt-call pillar.get _param:openstack_proxy_address

Open a web browser.

Paste the prx VIP and the default port 5601 to the web browser address field. No credentials are required.

Once you access the Kibana web UI, you must be redirected to the Kibana Logs analytics dashboard.
