###########################################
# Update mcp from 2018.1 tag to 2018.3.0
###########################################

1. update all tag 2018.1 to 2018.3.0 in cluster model

   infra/init.yml and cicd/control/init.yml


2. Enable the update pipelines in the model by adding the following class to the leader.yml file in the

  classes/cluster/${_param:cluster_name}/cicd/control/ directory:

      - system.jenkins.client.job.deploy.update.utils

3. Apply the change:

    salt -C 'I@jenkins:client' state.apply jenkins.client

4. 
