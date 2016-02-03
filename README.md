## OVERVIEW
This Ansible project was created by Data Curation Experts based on work for the Chemical Heritage Foundation, UC Santa Barbara, and Washington University in St. Louis. It builds a production-style Hydra application on CentOS 7 - on Amazon EC2, a Vagrant virtural machine, or a bare-bones server.

## Setup
Before you can use this project, you must:

1. Clone this repo to your local workstation using: `git clone https://github.com/acozine/sufia-centos.git`  * NOTE: this script runs on your local system 
   and connects to the server you want to configure, do not try to run it on the server directly*
1. Install [Ansible](http://docs.ansible.com/intro_installation.html) on your local system.
1. Continue with the Amaxon EC2, Vagrant, or SSH section depending on your server type

## EC2

To create an Amazon EC2 instance:

1. create new vars/main.yml files in the `launch_ec2` and `ec2` roles  
1. Add your organization's AWS credentials there
1. Create a new `vars/main.yml` file in the services role  
1. Override any default variables you wish to change there (we recommend overriding the postgresql database, user, and password settings)  
1. Run `ansible-playbook --private-key /path/to/your/keypair.pem ec2.yml` (if you encrypt your variables with ansible-vault, add `--ask-vault-pass`; if you are not using passwordless sudo, add `--ask-sudo-pass` or `-K`)  
1. If the playbook fails, you can restart it at a particular task with `ansible-playbook --private-key /path/to/your/keypair.pem ec2.yml --start-at-task='rolename | taskname'`  

## Vagrant

To use this project with [Vagrant](http://docs.vagrantup.com/v2/):

1. Create a Vagrant project  
1. Modify the Vagrantfile to use a Centos 7 box (`config.vm.box = `)  
1. Modify the Vagrantfile to use Ansible (see sample Vagrantfile for ideas)  
1. Be sure to point to the `vagrant.yml` file, which skips the `launch_ec2` and `ec2` roles
1. Clone this project as the `provisioning` sub-directory of your Vagrant project  
1. Run `vagrant up`  
1. If the playbook fails, you can restart it at a particular tasks by uncommenting the line in your Vagrantfile that begins `ansible.start_at_task` and changing the value to the task name  

## Generic server running SSH

To run the Ansible provisioning scripts against a minimal Centos 7 server via ssh:

1. Create an installation user with passwordless sudo access on your server.  We called ours `centos`.  If you use a different name, edit the remote\_user entry in vanilla.yml
1. Add the public key for your install user on the server in the users's .ssh/authorized\_keys file.  For help setting up public key authentication, see the [Centos Wiki](https://wiki.centos.org/HowTos/Network/SecuringSSH#head-9c5717fe7f9bb26332c9d67571200f8c1e4324bc) 
1. these scripts assume that you have a separate virtual drive available for the application installation at `/dev/sdc` (assuming `/dev/sda` is your main system drive and `/dev/sdb` is your swap drive).  
1. Edit the `hosts` file in this directory and replace the ip address for your Hydra application with your server's IP
1. Edit the `vars:` section of the `vanilla.yml` file and enter data for your system - see the commented lines
1. Run `ansible-playbook -i hosts vanilla.yml` from the root directory of this repo on your local machine
1. if the playbook fails, you can restart it at a particular task with `ansible-playbook -i hosts vanilla.yml --start-at-task='rolename | taskname'`  

## Next Steps

This project deploys your code with [Capistrano](http://capistranorb.com/). If your project doesn't already use 
Capistrano. Look to the Capistrano [getting started documentation](https://github.com/capistrano/capistrano#quick-start).

You'll need to update your Hydra application (your application repository) to define a new stage corresponding to the environment you've just set up.  
We usually give our stages names that help identify the servers they run on: e.g. demo, sandbox, stage, prod, etc.  Copy one of the existing stage files in your project (under `config/deploy`) to a new stage file.

 
In your stage file, set the Capistrano `:deploy_to` directory to match the Ansible housekeeping role's 
`project_name` variable. In `config/deploy.rb` and/or in `config/deploy/<yourenv>.rb` (depending on your project structure), 
add a line like:  

```
set :deploy_to, '/opt/sufia-project'
```
 
If you use the default value for `project_name` in the housekeeping role, this is the exact line you need.

1. In the main project repo for your rails application, create a new cap stage by copying one of the existing sage files in the `config/deploy` directory
1. Edit your new stage file
    a. Edit the line that says `set :stage, ` and add your stage name.  It's easiest to use the same name for the file and the stage.
    b. Edit (or add) the line that says `set :deploy_to, '/opt/...'` and set ... to the `project_name` you set in your Ansible YAML file varialbes
1. Run `cap {stage-name} deploy` where `{stage-name}` is the file and stage name you set in steps 1 & 2.  
1. After you run Capistrano the first time (and only the first time), restart Tomcat using `sudo systemctl restart tomcat`
1. Your application should be up and running! Check it in your browser.
   
