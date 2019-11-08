freehck.k8s_app
=========

Load application definition in kubernetes cluster

Description
-----------

This role allows to download YAML application definition and load it into kubernetes cluster.

Also it has a bunch of pre-downloaded definitions that you can use. Look available definition files under `files/` directory.

Pre-downloaded definitions:

- calico
- kubernetes-dashboard

Role Variables
--------------

`k8s_app_download_definition`: flag specifying we shall download definition from url or use pre-downloaded one (default is `false`, that means "not download, use pre-downloaded)

`k8s_app_definition_dir`: directory to store definition file (default is `/root`)

`k8s_app_definition_filename`: filename to store definition, must always be set explicitly

`k8s_app_definition_url`: URL of definition, this parameter is mandatory when you set `k8s_app_download_definition` to `true`

Example
-------

###### inventory

    k8s-node-0 ansible_host=10.118.19.10 k8s_is_master=true
    k8s-node-1 ansible_host=10.118.19.11
    k8s-node-2 ansible_host=10.118.19.12
    
    [k8s_cluster]
    k8s-node-0
    k8s-node-1
    k8s-node-2

###### group_vars/k8s_cluster.yml

    # common params
    k8s_ver: "1.16.2-00"
    k8s_node_ip: "{{ ansible_host }}"
    
    # k8s_base is an implicit dependency
    k8s_base_node_ip: "{{ k8s_node_ip }}"
    k8s_base_ver: "{{ k8s_ver }}"
    
    # k8s_init is an implicit dependency
    k8s_init_cidr: "192.168.0.0/16"
    k8s_init_node_ip: "{{ ansible_host }}"
    k8s_init_node_name: "{{ inventory_hostname }}"
    
    # this role configuration
    k8s_join_is_master: "{{ k8s_is_master | default('false') }}"


###### playbook.yml

    - hosts: k8s_cluster
      become: true
      roles:
        - role: freehck.k8s_base
        - role: freehck.k8s_init
          when: k8s_is_master | default(false)
        - role: freehck.k8s_join
        - role: freehck.k8s_app
          when: k8s_is_master | default(false)
          k8s_calico_ver: "v3.10"
          k8s_app_definition_filename: "calico-{{ k8s_calico_ver }}.yml"
          # --------------------
          # deploy from pre-downloaded service definition (uncomment it to test)
          k8s_app_download_definition: false
          # # --------------------
          # # download service definition from site (uncomment it to test)
          # k8s_app_download_definition: true
          # k8s_app_definition_url: |-
          #   https://docs.projectcalico.org/{{ k8s_calico_ver }}/manifests/calico.yaml

Tests
-----

This role provides you with a Vagrant-based test suite under `tests/` directory.

To test the role type following command in terminal:

    cd tests && make

Install
-------

This role can be installed from [Ansible Galaxy](https://galaxy.ansible.com/):

`ansible-galaxy install freehck.k8s_app`

License
-------

MIT

Author Information
------------------

Dmitrii Kashin, <freehck@freehck.ru>
