## Deploy OpenContrail manually ( Version: 4.*)


1. Log in to the Salt Master node.

2.  Run the following basic states to prepare the OpenContrail nodes:

        salt -C 'ntw* or nal*' saltutil.refresh_pillar
        salt -C 'I@opencontrail:database' saltutil.sync_all
        salt -C 'I@opencontrail:database' state.sls salt.minion,linux,ntp,openssh

3. Deploy and configure Keepalived and HAProxy:

        salt -C 'I@opencontrail:database' state.sls keepalived,haproxy

4. Deploy and configure Docker:

        salt -C 'I@opencontrail:database' state.sls docker.host

5. Create configuration files for OpenContrail:

        salt -C 'I@opencontrail:database' state.sls opencontrail exclude=opencontrail.client

6. Start the OpenContrail Docker containers:

        salt -C 'I@opencontrail:database' state.sls docker.client

7. Verify the status of the OpenContrail service:

        salt -C 'I@opencontrail:database' cmd.run 'doctrail all contrail-status'

    In the output, the services status should be active or backup.

8. Configure the OpenContrail resources:

        salt -C 'I@opencontrail:database and *01*' state.sls opencontrail.client

9. Apply the following states to deploy the OpenContrail vRouters:

        salt -C 'cmp*' saltutil.refresh_pillar
        salt -C 'I@opencontrail:compute' saltutil.sync_all
        salt -C 'I@opencontrail:compute' state.highstate exclude=opencontrail.client
        salt -C 'I@opencontrail:compute' cmd.run 'reboot'
        salt -C 'I@opencontrail:compute' state.sls opencontrail.client
