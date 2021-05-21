# Documentation for installing the staffeln service

### Setup Staffeln in OSA.
1. Create the ssh keypair in the OSA server. You can use following command to create the keypair.
```BASH
$ ssh-keygen
```
2. Copy the content of the public key and add it in the github authorized key section and enable sso.
3. Download the role from the github in roles directory is osa server.
```BASH
$ cd /opt/openstack-ansible/playbook
$ ansible-galaxy role install git+ssh://git@github.com/vexxhost/os_staffeln.git -p ./roles
```
4. Add the following section on `/etc/openstack_deploy/user_variables.yml`
```YAML
# Indicate if the repo is private
staffeln_repo_mode: private
# ssh key authorized to download package from git.
deploy_key: /root/.ssh/id_rsa
# Haproxy configuration
haproxy_extra_services:
  - service:
      haproxy_service_name: staffeln
      haproxy_backend_nodes: "{{ groups['staffeln_all'] | default([]) }}"
      haproxy_ssl: "{{ haproxy_ssl }}"
      haproxy_ssl_all_vips: true
      haproxy_port: "{{ haproxy_ssl | ternary(443,80) }}"
      haproxy_backend_port: 8808
      haproxy_redirect_http_port: 8808
      haproxy_balance_type: http
      haproxy_balance_alg: source
      haproxy_service_enabled: "{{ groups['staffeln_all'] is defined and groups['staffeln_all'] | length > 0 }}"
```
5. Add the following snippet in `/etc/openstack_deploy/env.d/staffeln.yml` to add the host in inventory.
```YAML
component_skel:
  staffeln:
    belongs_to:
      - staffeln_all


container_skel:
  staffeln_container:
    belongs_to:
      - cgm_containers
      - shared-infra_containers
    contains:
      - staffeln


physical_skel:
  cgm_containers:
    belongs_to:
      - all_containers
  staffeln_hosts:
    belongs_to:
      - hosts
```

6. You can run the staffeln-setup.yml playbook to install the staffeln service.