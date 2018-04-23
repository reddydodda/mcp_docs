## Kqueen 

1. refresh pillar and sync all

	salt '*' saltutil.sync_all 
	salt '*' saltutil.refresh_pillar


2.  Deploy the GlusterFS cluster

	salt -C 'I@glusterfs:server' state.sls glusterfs.server.service
	salt -C 'I@glusterfs:server and *01*' state.sls glusterfs.server.setup


3. Mount Gluster volumes from the KVM nodes:

	salt -C 'I@glusterfs:client and I@docker:host' state.sls glusterfs.client

4. Configure virtual IP and HAProxy balancing:

    	salt -C 'I@haproxy:proxy and I@docker:host' state.sls haproxy,keepalived

5. Start the CI/CD containers, for example, MySQL, Aptly, Jenkins, Gerrit, and others:

    salt -C 'I@docker:swarm:role:master' state.sls docker.client



Bugs:

https://mirantis.jira.com/browse/PROD-19570
https://mirantis.jira.com/browse/PROD-19571

