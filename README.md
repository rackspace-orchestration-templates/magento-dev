Description
===========

#### Development Site

A Single Linux server with
[Magento 1.9.2.0 Community Edition](http://www.magentocommerce.com/product/community-edition/)
installed with [nginx](http://nginx.org/en/), [PHP FPM](http://php-fpm.org/about/), and
[Percona](https://www.percona.com/software/mysql-database/percona-server).

This deployment is intended for development and test cases only. It is not
designed for hosting "live" sites and is not scalable.


Instructions
===========

#### Getting Started
If you're new to Magento Community Edition, the [Magento User
Guide](http://www.magentocommerce.com/resources/user-guide-download) will
step you through the process of configuring and managing your store. This
guide is free, but does require you to provide a valid email address to
receive it.

The [Magento Forum](http://www.magentocommerce.com/boards/) provides a place
to get answers to both simple and advanced questions regarding configuration
and management of Magento Community Edition.

#### Logging into Magento
To login, go to the URL listed above in a browser. If your DNS is not
pointing to this installation, you can add a line in your [hosts
file](http://www.rackspace.com/knowledge_center/article/how-do-i-modify-my-hosts-file)
to point your domain to the IP of this Cloud Server. Once you've done this,
restart your browser and navigate to the site. The backend can be accessed by
adding '/admin' to the end of the URL, and you can login with the credentials
provided above.

#### Logging in via SSH
The private key provided in the passwords section can be used to login as
root via SSH. We have an article on how to use these keys with [Mac OS X and
Linux](http://www.rackspace.com/knowledge_center/article/logging-in-with-a-ssh-private-key-on-linuxmac)
as well as [Windows using
PuTTY](http://www.rackspace.com/knowledge_center/article/logging-in-with-a-ssh-private-key-on-windows).

#### Details of Your Setup
This deployment was stood up using
[Ansible](http://www.ansible.com/home). Once the deployment is
up, Ansible will not run again, so it is safe to modify configurations.

[Nginx](http://nginx.org/en/) is used as the web server and listens on port
80 to handle web traffic. The configuration for your site can be
found in /etc/nginx/conf.d/. There will be a default site
configuration and a seperate one for your domain. Magento itself is
installed in /var/www/vhosts. You will find a directory with the name of
website you entered as a part of this deployment.

[PHP-FPM](http://php.net/manual/en/install.fpm.php) is used to handle
evaluation of all PHP-based pages. The configuration for this installation is
in /etc/php5/fpm/pools/magento.conf. By default, PHP-FPM is running as the
'magento' user, listens on 127.0.0.1:9001.

Object and session caching are handled by
[Redis](http://redis.io/). Redis is an in-memory, high performance key-value
and is used for caching data and persisting user session data. This enhances
the performance of your site by reducing expensive database calls. One persistent cache
is configured. You can find the configuration in /etc/redis/.

[Percona](https://www.percona.com/software/mysql-database/percona-server) database is
installed on the instance as the data store.

#### Magento Plugins
There are thousands of plugins that have been developed for Magento by the
developer community. [Magento
Connect](http://www.magentocommerce.com/magento-connect/) provides an easy
way to discover popular plugins that other users have found to be helpful.
Not all plugins are free, and we recommend only installing the plugins that
you need.

#### Additional Notes
**This deployment is meant for development and testing scenarios only and is not
intended for production use**.


Requirements
============
* A Heat provider that supports the following:
  * OS::Heat::RandomString
  * OS::Heat::SoftwareConfig
  * OS::Heat::SoftwareDeployment
  * OS::Nova::KeyPair
  * OS::Nova::Server
  * Rackspace::Cloud::BackupConfig
  * Rackspace::CloudMonitoring::Check
* An OpenStack username, password, and tenant id.
* [python-heatclient](https://github.com/openstack/python-heatclient)
`>= v0.2.8`:

```bash
pip install python-heatclient
```

We recommend installing the client within a [Python virtual
environment](http://www.virtualenv.org/).

Parameters
==========
Parameters can be replaced with your own values when standing up a stack. Use
the `-P` flag to specify a custom parameter.

* `magento_url`: Domain to use with Magento Site (Default: example.com)
* `magento_user`: Username for Magento admin (Default: admin)
* `magento_fname`: First name for Magento admin (Default: Joe)
* `magento_lname`: Last name for Magento admin (Default: User)
* `magento_email`: E-Mail for Magento admin (Default: admin@example.com)
* `magento_eula`: You must agree to the terms of the Magento Community Edition License 
* `magento_samples`: Include Magento Sample Data (Default: False)
* `server_flavor`: Flavor of Cloud Server to use for Magento (Default: 8 GB General Purpose v1)

Outputs
=======
Once a stack comes online, use `heat output-list` to see all available outputs.
Use `heat output-show <OUTPUT NAME>` to get the value of a specific output.

* `magento_login_user`: Magento Admin User 
* `magento_login_password`: Magento Admin Password 
* `magento_public_ip`: Server Public IP 
* `magento_admin_url`: Magento Admin URL 
* `magento_public_url`: Magento Public URL 
* `mysql_user`: Database User 
* `mysql_password`: Database Password 
* `ssh_private_key`: SSH Private Key 

For multi-line values, the response will come in an escaped form. To get rid of
the escapes, use `echo -e '<STRING>' > file.txt`. For vim users, a substitution
can be done within a file using `%s/\\n/\r/g`.
