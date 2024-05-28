# ansible-role-docker
Ansible role for installing Docker. 

The role installs a Docker daemon from Docker debian package repository. It also configures it in this way:
- log driver is set to journald
- IPv6 connectivity is enabled using
  [IPv6 Unique Local Addresses](https://en.wikipedia.org/wiki/Unique_local_address) and
  IPv6-to-IPv6 Network Address Translation (NAT66) analogous to IPv4 which is using 
  [IPv4 Private Addresses](https://en.wikipedia.org/wiki/Private_network) and NAT
- sets Docker's MTU size to the system setting

Optionally:
- creates a local bridge network for containers 
- holds docker packages from upgrading
- installs python3-docker package which is a requirement for Ansible docker modules
- installs [Portainer](https://hub.docker.com/r/portainer/portainer-ce) web app running on port 9000
- logs into container registries (provided by GitLab instances)
- creates docker volumes
- creates cron jobs

Role Variables
--------------

* **docker_packages** - list of packages to install and optionally hold, default is docker-ce,docker-ce-cli and containerd.io
* **docker_packages_hold** - whether to hold packages from upgrading, default is false
* **docker_registries** - list of triples (url,username,password) for docker login, default is empty
* **docker_local_network_create** - whether to create local bridge network, default is true
* **docker_local_network_name** - network name, default is "network"
* **docker_local_network_ipv6** - IPv6 connectivity of the local bridge network, default is true
* **docker_local_network_ipam_config** - override to set specific IP range and gateway 
* **docker_portainer_install** - whether to install Portainer, default is false
* **docker_portainer_admin_password** - administrator password, must be defined to install Portainer
* **docker_portainer_fullchain_file** - path to certificate and its chain, must be defined to install Portainer
* **docker_portainer_privkey_file** - path to certificate's private key, must be defined to install Portainer
* **docker_portainer_admin_password_file** - path to file with admin password, default is /etc/portainer_admin_password.txt
* **docker_portainer_admin_password_file_owner** - default is root
* **docker_portainer_admin_password_file_group** - default is root
* **docker_portainer_use_certbot_certificates** - whether to mount certbot's directories to Portainer, default is false
* **docker_cron_jobs** - list of cron job definitions, default is empty
* **docker_cron_jobs_force** - boolean whether to run cron jobs immediately, default is false
* **docker_volumes** - list of volume names to create, default is empty

Examples of Playbooks
----------------

```yaml
- hosts: all 
  roles:
     - role: cesnet.docker
       vars:
         docker_packages_hold: true
         docker_registries:
           - url: registry.gitlab.ics.muni.cz:443
             username: gitlab-read-only-deploy-token-perun-all
             password: BgdJHJ_gsmyqp3se5WRx
         docker_local_network_name: perun_net
         docker_portainer_install: true
         docker_portainer_admin_password_file: "{{ perun_certs_dir }}/portainer_admin_password.txt"
         docker_portainer_admin_password: "{{ perun_portainer_admin_password }}"
         docker_portainer_admin_password_file_owner: root
         docker_portainer_admin_password_file_group: perun
         docker_portainer_use_certbot_certificates: "{{ perun_use_certbot_certificates }}"
         docker_portainer_fullchain_file: "{{ perun_certificate_fullchain_file }}"
         docker_portainer_privkey_file: "{{ perun_certificate_key_file }}"
```
