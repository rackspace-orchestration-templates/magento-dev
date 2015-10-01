heat_template_version: 2015-10-15

description: |
  #### Development Site

  A Single Linux server with
  [Magento 1.9.2.0 Community Edition](http://www.magentocommerce.com/product/community-edition/)
  installed with [nginx](http://nginx.org/en/), [PHP FPM](http://php-fpm.org/about/), and
  [Percona](https://www.percona.com/software/mysql-database/percona-server).

  This deployment is intended for development and test cases only. It is not
  designed for hosting "live" sites and is not scalable.


parameter_groups:

- label: Server Settings
  parameters:
  - flavor

- label: Magento Settings
  parameters:
  - eula
  - domain_name
  - domain_sftp_user
  - include_samples

parameters:

  eula:
    type: boolean
    label: Agree to terms?
    description: You must agree to the terms of the Magento Community Edition License
    constraints:
    - allowed_values:
      - True
      description: |
        You must agree to the Magento Community Edition License
        which can be found here: http://opensource.org/licenses/osl-3.0.php

  domain_name:
    type: string
    default: example.com
    label: Site Domain
    description: Magento site domain name

  domain_sftp_user:
    type: string
    default: magento_sftp
    label: Domain SFTP user
    description: Domain and PHP-FPM user

  flavor:
    type: string
    default: 8 GB Performance
    constraints:
    - allowed_values:
      - 8GB Standard Instance
      - 15GB Standard Instance
      - 30GB Standard Instance
      - 8 GB Performance
      - 15 GB Performance
      - 30 GB Performance
      - 60 GB Performance
      - 90 GB Performance
      - 120 GB Performance
    label: Server Size
    description: Amount of RAM for the Magento server

  include_samples:
    type: boolean
    default: False
    label: Include Sample Data?
    description: Include Magento sample store data

resources:

  server_pw:
    type: OS::Heat::RandomString

  mysql_pw:
    type: OS::Heat::RandomString

  mysql_root_pw:
    type: OS::Heat::RandomString

  sftp_password:
    type: OS::Heat::RandomString

  magento_admin_pass:
    type: OS::Heat::RandomString

  upload_role_config:
    type: OS::Heat::SoftwareConfig
    properties:
      group: script
      outputs:
      - name: results
      config: |
        #!/bin/bash
        set -e
        git clone --progress https://github.com/rackspace-orchestration-templates/ansible-roles.git /etc/ansible/roles > $heat_outputs_path.results 2>&1

  magento_config:
    type: OS::Heat::SoftwareConfig
    properties:
      group: ansible
      config: |
        ---
        - name: Install and configure Magento
          hosts: localhost
          connection: local
          roles:
          - percona
          - nginx
          - php
          - php-fpm
          - redis
          - mysqldb
          - magento-ce

  deploy_roles:
    type: OS::Heat::SoftwareDeployment
    properties:
      signal_transport: TEMP_URL_SIGNAL
      config:
        get_resource: upload_role_config
      server:
        get_resource: magento_server

  deploy_magento:
    type: OS::Heat::SoftwareDeployment
    depends_on:
    - deploy_roles
    properties:
      signal_transport: TEMP_URL_SIGNAL
      input_values:
        tz: "America/Chicago" 
        webserver: nginx
        vhost_domain: { get_param: domain_name }
        vhost_aliases: "www.{{ vhost_domain }}"
        vhost_user: nginx
        vhost_user_uid: 1010
        vhost_user_shell: /sbin/nologin
        document_root: "/var/www/vhosts/{{ vhost_domain }}/httpdocs"
        domain_sftp_user: { get_param: domain_sftp_user }
        domain_sftp_password: { get_attr: [ sftp_password, value ] }
        placeholder_content: { get_param: include_samples }
        redis_host: localhost
        redis_instances:
        - { name: 'cache', port: '6381', persistent: 'TRUE', maxmemory: '1gb' }
        magento_db_name: magento
        magento_db_user: magento
        magento_db_password: { get_attr: [ mysql_pw, value ] }
        mysql_host_ip: localhost
        mysql_root_pw: { get_attr: [ mysql_root_pw, value ] }
        mysql_db_name: "{{ magento_db_name }}"
        mysql_db_user: "{{ magento_db_user }}"
        mysql_db_password: "{{ magento_db_password }}"
        magento_admin_fname: Magento Dev
        magento_admin_lname: Admin
        magento_admin_email:
          str_replace:
            template: "admin@domain"
            params: { domain: { get_param: domain_name } }
        magento_admin_user: devadmin
        magento_admin_pass: { get_attr: [ magento_admin_pass, value ] }
      config:
        get_resource: magento_config
      server:
        get_resource: magento_server

  bootstrap:
    type: https://raw.githubusercontent.com/rackspace-orchestration-templates/software-config-bootstrap/master/bootconfig_yum_ansible.yaml

  access_key:
    type: OS::Nova::KeyPair
    properties:
      name:
        str_replace:
          template: stack-keypair
          params:
            stack: { get_param: "OS::stack_name" }
      save_private_key: True

  magento_server:
    type: OS::Nova::Server
    properties:
      name:
        str_replace:
          template: stack-magento-server
          params:
            stack: { get_param: "OS::stack_name" }
      image: CentOS 7 (PVHVM)
      key_name: { get_resource: access_key }
      flavor: { get_param: flavor }
      software_config_transport: POLL_TEMP_URL
      config_drive: true
      user_data_format: SOFTWARE_CONFIG
      user_data: { get_attr: [ bootstrap, config ] }
      metadata:
        rax-heat: { get_param: "OS::stack_id" }

outputs:

  ssh_pub_key:
    description: SSH Public Key
    value:
      get_attr: [ access_key, public_key ]

  ssh_priv_key:
    description: SSH Private Key
    value:
      get_attr: [ access_key, private_key ]

  server_pub_ip:
    description: Server Public IP
    value:
      get_attr: [ magento_server, accessIPv4 ]

  magento_domain:
    description: Magento Domain
    value:
      get_param: domain_name

  admin_url:
    description: Admin URL
    value:
      str_replace:
        template: "https://domain/admin"
        params:
          domain: { get_param: domain_name }

  domain_sftp_user:
    description: SFTP User
    value: magento_sftp

  domain_sftp_password:
      description: SFTP User Password
      value:
        get_attr: [ sftp_password, value ]

  magento_url:
    description: Store URL
    value:
      str_replace:
        template: "http://domain"
        params:
          domain: { get_param: domain_name }

  magento_db_name:
    description: Magento database name
    value: magento

  magento_db_user:
    description: Magento database user
    value: magento

  magento_db_password:
    description: Magento database user password
    value:
      get_attr: [ mysql_pw, value ]

  magento_db_root_pw:
    description: Magento database root user password
    value:
      get_attr: [ mysql_root_pw, value ]

  magento_admin_user:
    description: Magento Admin User
    value: devadmin

  magento_admin_password:
    description: Magento Admin Password
    value:
      get_attr: [ magento_admin_pass, value ]
