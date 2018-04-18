# Configure LDAP integration with MCP
--------------------------------------
1. In your Git project repository, open the cluster/<cluster_name>/openstack/ directory of your cluster model.

    In this directory, create the ldap.yml file.

    Create a configuration for the LDAP integration in this file.

    Example:

parameters:
  keystone:
    server:
    service_name: apache2
    domain:
        <DOMAIN NAME>:
        description: ""
        backend: ldap
        identity:
            backend: ldap
        assignment:
            backend: sql
        ldap:
            url: ldap://<LDAP ADDRESS>
            bind_user: CN=<UserName>,OU=<OU-name>,DC=<DomainName>,DC=<DomainExtension>
            query_scope: sub
            page_size: 1000
            password: <LDAP PASSWORD>
            suffix: DC=<DomainName>,DC=<DomainExtension>
            user_tree_dn: DC=<DomainName>,DC=<DomainExtension>
            group_tree_dn: DC=<DomainName>,DC=<DomainExtension>
            user_objectclass: person
            user_id_attribute: sAMAccountName
            user_name_attribute: sAMAccountName
            user_pass_attribute: userPassword
            user_enabled_attribute: userAccountControl
            user_mail_attribute: mail
            group_objectclass: ""
            group_id_attribute: sAMAccountName
            group_name_attribute: cn
            group_member_attribute: member
            group_desc_attribute: cn
            filter:
            user: "(&(&(objectClass=person)(uidNumber=*))(unixHomeDirectory=*))"
            group: ""

2. In cluster.yml, include the previously created class to the bottom of the classes section:

classes:
  ...
  cluster.<cluster_name>.openstack.ldap
  cluster.<cluster_name>
parameters:
  ...

3. Enforce the Keystone update using the Jenkins Deploy - update service(s) config pipeline or directly using Salt:

  salt 'ctl01*' state.sls keystone -b1 

4. Add parameters for Horizon to
      cluster/<cluster_name>/openstack/proxy.yml:

    parameters:
      horizon:
        server:
          multidomain: true

5. Enforce the Keystone update using the Jenkins

      Deploy - update service(s) config - pipeline or

   Directly using Salt:

      salt -C 'I@keystone:server and *01*' state.sls keystone
      salt -C 'I@keystone:server and not *01*' state.sls keystone
      salt -C 'I@horizon:server' state.sls horizon


6. Verify the LDAP integration:

  source /roo/keystonercv3
  openstack user list --domain <your_domain>

7. Grant the admin role to a specific user:

  Obtain the user ID:

  openstack user list --domain <your_domain> | grep <user_name>
| <user_id> | <user_name>  |

8. Set the admin role:

  openstack role add --user <user_id> admin --domain <your_domain>


#  To enable PAM authentication for Host OS:
--------------------------------------------------

1. Open the Git project repository with your cluster model.

2. Create the cluster/<cluster_name>/infra/auth/ldap.yml file.

3. Create a configuration for your LDAP server in this file.

Example:

parameters:
  linux:
    system:
      auth:
        enabled: true
        ldap:
          enabled: true
          binddn: CN=<UserName>,OU=<OU-name>,DC=<DomainName>,DC=<DomainExtension>
          bindpw: <Password>
          uri: ldap://<LDAP URL>
          base: DC=<DomainName>,DC=<DomainExtension>
          ldap_version: 3
          pagesize: 1000
          referrals: "off"
          ##You can also setup grouping, mapping, and filtering using these parameters.
          filter:
            passwd: (&(&(objectClass=person)(uidNumber=*))(unixHomeDirectory=*))
            shadow: (&(&(objectClass=person)(uidNumber=*))(unixHomeDirectory=*))
            group:  (&(objectClass=group)(gidNumber=*))
          map:
            passwd:
              uid: sAMAccountName
              homeDirectory: unixHomeDirectory
              gecos: displayName
              loginShell: '"/bin/bash"'
            shadow:
              uid: sAMAccountName
              shadowLastChange: pwdLastSet

4. In cluster/<cluster_name>/openstack/cluster.yml, include the previously created class to the bottom of the classes section:

classes:
  ...
  cluster.<cluster_name>.infra.auth.ldap
  cluster.<cluster_name>
parameters:
  ...

5. Enforce the linux.system update using the Jenkins Deploy - update service(s) config pipeline or directly using Salt:

  salt '<target_node>*' state.sls linux.system



  10.101.0.90
