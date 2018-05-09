## Upgrade from once version to another
--------------------------------------------

1. Change the build id in follwoing location

    a. cluster/<cluster_name>/infra/init.yml

        openstack_version: ocata
        apt_mk_version: 2018.4.0

    b. cluster/<cluster_name>/cicd/control/init.yml

        openstack_version: ocata
        apt_mk_version: 2018.4.0

2. Update mk-pipelines and pipeline-library using jenkins

3. Verify that the following class is present in the      

    cluster/cicd/control/leader.yml file of your Reclass model:

        classes:
          - system.jenkins.client.job.deploy.update

4. Mkae sure take backup of any custom changes to classes/system

        git diff > 2018.4.0.patch

5. Update the classes/system to updating tag at classes/system

      git checkout 2018.4.0

6. apply the patch to classes/system

     git apply 2018.4.0.patch

7. Update Jenkins pipelines:

    salt -C 'I@jenkins:client' state.sls jenkins.client

8. Run the Deploy - update cloud pipeline in the Jenkins web UI.



## Bugs
-----------------------------

1. MaaS package bug

https://mirantis.jira.com/browse/PROD-19812

2. cfg01 node reboot pipeline failur

https://mirantis.jira.com/browse/PROD-19844

3. MCP_openStack and mk_openstack repo fix

https://mirantis.jira.com/browse/PROD-19845
