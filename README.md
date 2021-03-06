# sat6-deployer
This repository contains roles and playbooks to automatically configure a Satellite 6 server based on variables provided by the user.

## Prerequisites
Install a minimal RHEL 7 according to the Satellite 6 documentation: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.4/html/installing_satellite_server_from_a_connected_network/

### Register with subscription-manager
I have intentionally left the registration steps manual because this may differ a lot between different environments. The following instructions can be seen as an example, but may not be exactly the same for you.

Register with subscription-manager:
```bash
# subscription-manager register
```

Find an available Satellite subscription you want to use and attach it with its pool ID:
```bash
# subscription-manager list --available
# subscription-manager attach --pool <pool ID of your Satellite subscription>
# subscription-manager repos \
--disable="*"
--enable=rhel-7-server-rpms \
--enable=rhel-server-rhscl-7-rpms \
--enable=rhel-7-server-satellite-6.4-rpms \
--enable=rhel-7-server-satellite-maintenance-6-rpms \
--enable=rhel-7-server-ansible-2.6-rpms
```
https://access.redhat.com/documentation/en-us/red_hat_satellite/6.4/html/installing_satellite_server_from_a_connected_network/installing_satellite_server#registering_subscription_management_satellite

### Clone this repository
```bash
# yum -y install git
# git clone https://github.com/ture-karlsson/sat6-deployer.git
# cd sat6-deployer/
```

## Usage

### hosts file
Make sure your Satellite hostname is the only host in the "satellite" group in the hosts file. Replace "sat6" with your hostname, e.g:
```bash
# sed -i "s/sat6.example.com/$(hostname)/" hosts
# cat hosts
[satellite]
your-satellite.hostname.com
```

### Add your manifest file to the files directory
```bash
# ls files/
manifest.zip
```

### vars/sat-vars.yml
The variable file vars/sat-vars.yml contains all the variables that are used for the installation and configuration of the Satellite server. Make sure to read it through and update it to fit your needs. This lets you define exactly what repositories to synchronize and all other configuration that will be set up in the Satellite.

### Run sat6-install.yml playbook

I have added a simple playbook called sat6-install.yml that can install the Satellite software for you. However, I recommend doing this step as well manually, since it differs a lot between different deployments. It is recommended to read through satellite-installer --scenario satellite --help to see what options that is needed in your environment.

However, if you want to use sat6-install.yml, it can be run with the following command.
```bash
# ansible-playbook -i hosts sat6-install.yaml
```

It may be wise to reboot your Satellite at this stage if yum updated packages that require reboot.

### Run sat6-configure.yaml playbook
If the installation was successful, it is time to put some content in your Satellite. This is what takes the most time, but this is also where this automation saves you a lot of time. Again, make sure vars/sat-vars.yml looks as you expect it to. All configuration are based on the repositories, content views and host groups etc that are defined in there. The playbook can be run in different phases to configure only one or a couple types of objects by using the defined tags, e.g.

```bash
# ansible-playbook -i hosts sat6-configure.yml --tags settings
# ansible-playbook -i hosts sat6-configure.yml --tags repositories,content-views,activation-keys
# ansible-playbook -i hosts sat6-configure.yml --tags host-groups
```
or everything at once:
```bash
# ansible-playbook -i hosts sat6-configure.yml
```

Have a look around in the GUI and see if all objects were created according to your expectations.

## Issues?
Please report issues and/or questions. Pull requests are very welcome.

## Future improvements
This is very dependant on the command module executing hammer with is far from optimal. My plan is to start using https://github.com/theforeman/foreman-ansible-modules instead. Again, PRs are welcome.
