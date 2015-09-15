## OVERVIEW
This Ansible project was created by Data Curation Experts based on work for the Chemical Heritage Foundation, UC Santa Barbara, and Washington University in St. Louis. It builds a production-style Hydra head on CentOS 7 - on Amazon EC2, on a Vagrant vm, or on a bare-bones box in your server environment.

## Setup
Before you can use this project, you must:

1. clone this repo to your local workstation using: `git clone https://github.com/acozine/sufia-centos.git`  *NOTE: this script runs on your local system 
   and connects to the server you want to configure, do not try to run it on the server directly*
2. install [Ansible](http://docs.ansible.com/intro_installation.html) on your local system.
3. visit the [Oracle downloads page](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
4. accept Oracle's License Agreement for Java 8
5. download the Java SE Development Kit 8u51 as a tarball from Oracle - the one you want is `Linux x64      165.25 MB        jdk-8u51-linux-x64.tar.gz`
6. place the file in roles/hydra-stack/files/jdk-8u51-linux-x64.tar.gz
7. continue with the EC2, Vagrant, or SSH section depending on your server type

## EC2

To create an ec2 instance:

1. create new vars/main.yml files in the launch_ec2 and ec2 roles  
2. add your organization's AWS credentials there
3. create a new vars/main.yml file in the services role  
4. override any default variables you wish to change there (we definitely recommend overriding the postgresql database, user, and password settings)  
5. run `ansible-playbook --private-key /path/to/your/keypair.pem ec2.yml` (if you encrypt your variables with ansible-vault, add `--ask-vault-pass`; if you are not using passwordless sudo, add `--ask-sudo-pass` or `-K`)  

## Vagrant

To use this project with [Vagrant](http://docs.vagrantup.com/v2/):

1. create a Vagrant project  
2. modify the Vagrantfile to use a centos 7 box ( config.vm.box = )  
3. modify the Vagrantfile to use Ansible (see sample Vagrantfile for ideas)  
4. be sure to point to the vagrant.yml file, which skips the launch_ec2 and ec2 roles
5. clone this project as the `provisioning` sub-directory of your Vagrant project  
6. run `vagrant up`


## Generic server running SSH

To run the Ansible provisioning scripts against a minimal Centos 7 server via ssh:

1. edit the `hosts` file in this directory and replace the ip address for hydra-head with your server's IP
2. create an installation user with passwordless sudo access on your server.  We called ours `centos`.  If you use a different name, edit the remote\_user entry in vanilla.yml
3. add the public key for your install user on the server in the users's .ssh/authorized\_keys file 
4. edit the `vars:` section of the vanilla.yml file and enter data for your system - see the commented lines
5. run `ansible-playbook vanilla.yml` from the root directory of this repo on your local machine

## Next Steps

This project expects your code to be deployed with [Capistrano](http://capistranorb.com/). In your Hydra head (the codebase you're deploying), set the Capistrano `:deploy_to` directory to match the housekeeping role's `project_name` variable. If you use the default value for `project_name` in the housekeeping role, you should use 
```
set :deploy_to, '/opt/sufia-project'
```
in `config/deploy.rb` and/or in `config/deploy/<yourenv>.rb`  
