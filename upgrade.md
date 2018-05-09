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



## Upgrade MCP Drive Train
----------------------------

1. Enable the update pipelines in the model by adding the following class to the leader.yml file in the
 classes/cluster/${_param:cluster_name}/cicd/control/ directory:

    - system.jenkins.client.job.deploy.update.utils

2. Apply the change:

    salt -C 'I@jenkins:client' state.apply jenkins.client

3. Log in as Administrator or refresh your Jenkins web UI.

      MCP DriveTrain component	       Pipeline

      System Reclass model	           Deploy - Update reclass metadata
      Salt formulas	                   Deploy - Update salt master formulas
      Jenkins pipelines	               Deploy - Update jenkins master jobs

4.

## Update MCP cluster
--------------------------

1. This pipeline helps with

      Delivering hot fixes to source code of OpenStack, Kubernetes, or MCP Control Plane.
      Updating packages for an OpenStack service.
      Applying security patches to operating system components and packages.

2. Change service configuration by running below Job

      Update Service(s) Configuration

3. Update service packages by running below Jobs

      Update System Package(s)

4. Add a service to MCp cluster

      Update Service(s) Configuration

5.


## Upgrade an MCP cluster
-----------------------------

1. Major upgrades enable:

    Delivering new features of MCP Control plane.
    Upgrading between OpenStack releases.
    Upgrading host and guest operating systems to new versions including kernels.
    Upgrading the OpenStack compute nodes.
    Updating the LCM platform.
    Upgrading the OpenStack OVS gateway nodes.
    Upgrading the OpenContrail nodes to 3.2 version.
    Upgrading the OpenContrail nodes to 4.x version.

2.


## Bugs
-----------------------------

1. MaaS package bug

https://mirantis.jira.com/browse/PROD-19812

2. cfg01 node reboot pipeline failur

https://mirantis.jira.com/browse/PROD-19844

3. MCP_openStack and mk_openstack repo fix

https://mirantis.jira.com/browse/PROD-19845
