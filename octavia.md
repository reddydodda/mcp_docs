# Configure load balancing with OpenStack Octavia
--------------------------------------------------

1. To configure Octavia as a Neutron LBaaS back end:

    a. Log in to the Salt Master node.

        Open the cluster/<NAME>/openstack/control.yml file for editing.

        Include the system.neutron.control.openvswitch.octavia class to the file.

        Example:

          classes:
            - system.neutron.control.openvswitch.octavia

    b. Select from the following options:

        If you are performing the initial deployment of your environment, proceed with the further environment configuration.

        If you are making changes to an existing environment, re-run the salt configuration on the Salt Master node to apply changes:

        salt '*' saltutil.refresh_pillar
        salt -C 'I@neutron:server' state.sls neutron -b 1

2. To enable Octavia:

    a. Log in to the Salt Master node.

        Include the following class to cluster/<NAME>/openstack/database.yml:

          - system.galera.server.database.octavia

        Include the following class to cluster/<NAME>/openstack/control_init.yml:

          - system.keystone.client.service.octavia

        Include the following classes to cluster/<NAME>/infra/config.yml:

          - system.glance.client.image.octavia
          - system.nova.client.service.octavia
          - system.neutron.client.service.octavia

        Include the following class to cluster/<NAME>/openstack/control.yml:

          - system.octavia.api.cluster

        The command above configures an Octavia API cluster to run on the OpenStack controller nodes.

    b. Alternatively, if you want to run a single instance of Octavia API, include the following class instead of the class mentioned in the previous step:

          - system.octavia.api.single

    c. Configure the Octavia management services (Controller Worker, Health Manager, and Housekeeping) in cluster/<NAME>/infra/config.yml:

      If you run OpenStack gateway services in a cluster, include the following class:

        - system.reclass.storage.system.openstack_gateway_single_octavia

        before

        - system.reclass.storage.system.openstack_gateway_cluster

      If you run OpenStack gateway services in a single mode, include the following class:

        - system.reclass.storage.system.openstack_gateway_single_octavia

        before

        - system.reclass.storage.system.openstack_gateway_single

      This would configure Octavia management services (Controller Worker, Health Manager, and Housekeeping) to run on one of the gateway nodes (gtw01 by default).


    c. Verify that the cluster/<NAME>/openstack/octavia_manager.yml class exists and contains import of the following classes as well as a private key that will be used to log in into amphorae. For example:

      classes:
      - system.octavia.manager.single
      - system.salt.minion.ca.octavia_ca
      - system.salt.minion.cert.octavia.amphora_client
      parameters:
        _param:
          octavia_private_key: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEpAIBAAKCAQEAtjnPDJsQToHBtoqIo15mdSYpfi8z6DFMi8Gbo0KCN33OUn5u
            OctbdtjUfeuhvI6px1SCnvyWi09Ft8eWwq+KwLCGKbUxLvqKltuJ7K3LIrGXkt+m
            qZN4O9XKeVKfZH+mQWkkxRWgX2r8RKNV3GkdNtd74VjhP+R6XSKJQ1Z8b7eHM10v
            6IjTY/jPczjK+eyCeEj4qbSnV8eKlqLhhquuSQRmUO2DRSjLVdpdf2BB4/BdWFsD
            YOmX7mb8kpEr9vQ+c1JKMXDwD6ehzyU8kE+1kVm5zOeEy4HdYIMpvUfN49P1anRV
            2ISQ1ZE+r22IAMKl0tekrGH0e/1NP1DF5rINMwIDAQABAoIBAQCkP/cgpaRNHyg8
            ISKIHs67SWqdEm73G3ijgB+JSKmW2w7dzJgN//6xYUAnP/zIuM7PnJ0gMQyBBTMS
            NBTv5spqZLKJZYivj6Tb1Ya8jupKm0jEWlMfBo2ZYVrfgFmrfGOfEebSvmuPlh9M
            vuzlftmWVSSUOkjODmM9D6QpzgrbpktBuA/WpX+6esMTwJpOcQ5xZWEnHXnVzuTc
            SncodVweE4gz6F1qorbqIJz8UAUQ5T0OZTdHzIS1IbamACHWaxQfixAO2s4+BoUK
            ANGGZWkfneCxx7lthvY8DiKn7M5cSRnqFyDToGqaLezdkMNlGC7v3U11FF5blSEW
            fL1o/HwBAoGBAOavhTr8eqezTchqZvarorFIq7HFWk/l0vguIotu6/wlh1V/KdF+
            aLLHgPgJ5j+RrCMvTBoKqMeeHfVGrS2udEy8L1mK6b3meG+tMxU05OA55abmhYn7
            7vF0q8XJmYIHIXmuCgF90R8Piscb0eaMlmHW9unKTKo8EOs5j+D8+AMJAoGBAMo4
            8WW+D3XiD7fsymsfXalf7VpAt/H834QTbNZJweUWhg11eLutyahyyfjjHV200nNZ
            cnU09DWKpBbLg7d1pyT69CNLXpNnxuWCt8oiUjhWCUpNqVm2nDJbUdlRFTzYb2fS
            ZC4r0oQaPD5kMLSipjcwzMWe0PniySxNvKXKInFbAoGBAKxW2qD7uKKKuQSOQUft
            aAksMmEIAHWKTDdvOA2VG6XvX5DHBLXmy08s7rPfqW06ZjCPCDq4Velzvgvc9koX
            d/lP6cvqlL9za+x6p5wjPQ4rEt/CfmdcmOE4eY+1EgLrUt314LHGjjG3ScWAiirE
            QyDrGOIGaYoQf89L3KqIMr0JAoGARYAklw8nSSCUvmXHe+Gf0yKA9M/haG28dCwo
            780RsqZ3FBEXmYk1EYvCFqQX56jJ25MWX2n/tJcdpifz8Q2ikHcfiTHSI187YI34
            lKQPFgWb08m1NnwoWrY//yx63BqWz1vjymqNQ5GwutC8XJi5/6Xp+tGGiRuEgJGH
            EIPUKpkCgYAjBIVMkpNiLCREZ6b+qjrPV96ed3iTUt7TqP7yGlFI/OkORFS38xqC
            hBP6Fk8iNWuOWQD+ohM/vMMnvIhk5jwlcwn+kF0ra04gi5KBFWSh/ddWMJxUtPC1
            2htvlEc6zQAR6QfqXHmwhg1hP81JcpqpicQzCMhkzLoR1DC6stXdLg==
            -----END RSA PRIVATE KEY-----

      The private key is saved to the /etc/octavia/.ssh/octavia_ssh_key file on the Octavia manager node.

      Note

        To generate an SSH key-pair, run:

          ssh-keygen -b 2048 -t rsa -N "" -f ~/.ssh/octavia_ssh_key

    d. Verify that the following Octavia parameters are configured in cluster/<NAME>/openstack/init.yml. For example:

parameters:
  _param:
     octavia_version: ${_param:openstack_version}
     octavia_service_host: ${_param:openstack_control_address}
     mysql_octavia_password: <db_password>
     keystone_octavia_password: <keystone_password>
     amp_flavor_id: <amphora-flavor-id>
     octavia_hm_bind_ip: 192.168.0.12
     octavia_public_key: |
       ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2Oc8MmxBOgcG2ioijXmZ1J
       il+LzPoMUyLwZujQoI3fc5Sfm45y1t22NR966G8jqnHVIKe/JaLT0W3x5bCr4
       rAsIYptTEu+oqW24nsrcsisZeS36apk3g71cp5Up9kf6ZBaSTFFaBfavxEo1X
       caR0213vhWOE/5HpdIolDVnxvt4czXS/oiNNj+M9zOMr57IJ4SPiptKdXx4qW
       ouGGq65JBGZQ7YNFKMtV2l1/YEHj8F1YWwNg6ZfuZvySkSv29D5zUkoxcPAPp
       6HPJTyQT7WRWbnM54TLgd1ggym9R83j0/VqdFXYhJDVkT6vbYgAwqXS16SsYf
       R7/U0/UMXmsg0z root@cfg01

       Note

       The parameter octavia_public_key should contain a public key generated in the previous step. In our example, it is taken from the ~/.ssh/octavia_ssh_key.pub file.

    e. Optional. Override the default Octavia parameters in cluster/<NAME>/openstack/octavia_manager.yml:

parameters:
  octavia:
    manager:
      certificates:
        ca_private_key: '/etc/octavia/certs/private/cakey.pem'
        ca_certificate: '/etc/octavia/certs/ca_01.pem'
      controller_worker:
        amp_flavor_id: ${_param:amp_flavor_id}
        amp_image_tag: amphora
        amp_ssh_key_name: octavia_ssh_key
        loadbalancer_topology: 'SINGLE'
      haproxy_amphora:
        client_cert: '/etc/octavia/certs/client.pem'
        client_cert_key: '/etc/octavia/certs/client.key'
        client_cert_all: '/etc/octavia/certs/client_all.pem'
        server_ca: '/etc/octavia/certs/ca_01.pem'
      health_manager:
        bind_ip: ${_param:octavia_hm_bind_ip}
        heartbeat_key: 'insecure'
      house_keeping:
        spare_amphora_pool_size: 0


3. Select from the following options:

  a. If you are performing the initial deployment of your environment, proceed with further environment configuration.

    After deploying the OpenStack core services, apply the following states:

    1. Upload an amphora image to Glance:

      salt -C 'I@glance:client' state.sls glance.client

    2. Create an amphora flavor and a key pair in Nova:

      salt -C 'I@nova:client' state.sls nova.client

      This state expects you to provide an SSH key that is used to create a key pair.

    3. Create Neutron resources for Octavia:

      salt -C 'I@neutron:client' state.sls neutron.client

      This state creates security groups and rules for the amphora instances and Health Manager, a management network with a subnet for Octavia, and a port for Health Manager.

    4. Update the Salt mine:

      salt-call mine.update '*'

    5. Deploy the Octavia services:

      salt -C 'I@octavia:api' state.sls octavia
      salt -C 'I@octavia:manager' state.sls octavia

    6. Generate certificates for the Octavia controller-amphora communication:

      salt -C 'I@octavia:manager' state.sls salt.minion.ca
      salt -C 'I@octavia:manager' state.sls salt.minion.cert

  b. If you are making changes to an existing environment, re-run the following states:

    1. Add the configured Octavia roles to the corresponding nodes:

      salt-call state.sls reclass.storage

      Refresh pillars:

      salt '*' saltutil.refresh_pillar

    2. Update the Salt Minion configuration:

      salt-call state.sls salt.minion.service

    3. Create the Octavia database:

      salt -C 'I@galera:master' state.sls galera
      salt -C 'I@galera:slave' state.sls galera -b 1

    4. Configure HAProxy for Octavia API:

      salt -C 'I@haproxy:proxy' state.sls haproxy

   5. Create an Octavia user and endpoints in Keystone:

      salt -C 'I@keystone:client' state.sls keystone.client

   6. Upload an amphora image to Glance:

      salt -C 'I@glance:client' state.sls glance.client

   7. Create an amphora flavor and a key pair in Nova:

      salt -C 'I@nova:client' state.sls nova.client

      This state expects you to provide an SSH key that is used to create a key pair.

   8. Create the Neutron resources for Octavia:

      salt -C 'I@neutron:client' state.sls neutron.client

      This state creates security groups and rules for amphora instances and Health Manager, a management network with a subnet for Octavia, and a port for Health Manager.

   9. Update the Salt mine:

      salt-call mine.update '*'

  10. Deploy Octavia services:

      salt -C 'I@octavia:api' state.sls octavia
      salt -C 'I@octavia:manager' state.sls octavia

  11. Generate certificates for the Octavia controller-amphora communication:

      salt -C 'I@octavia:manager' state.sls salt.minion.ca
      salt -C 'I@octavia:manager' state.sls salt.minion.cert
