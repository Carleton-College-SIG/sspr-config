# sspr-config
Carleton SSPR configuration

## SSPR-Configuration Introduction

Self Service Password Reset (SSPR) is a web-based password management service. You can deploy Self Service Password Reset to any web server or application server that supports a web archive. By eliminating the user dependencies on help desk assistance for password reset and other tasks, SSPR will reduce the workload of Carleton ITS help desk.

## Current Main Features
The following features are currently available for all users on SSPR.
- Landing page
- Change Password Module
- Setup Mobile App Authentication Module
- Reset Forgotten Passwords
- Recover Forgotten Username
- Editing alternate email addresses

The Help Desk Module allows ITS help desk workers/supervisors to do the following
- Assist in resetting passwords for users
- Clearing intruder lockout
- Unlocking user accounts

People assigned to administrative roles will be able to take the following actions
- Create Password Policies
- Check the status of SSPR through the configuration manager
- Administrators can customize SSPR through the configuration editor


## Before you begin

1. Clone this repository into a directory on the control host.
1. Create the file .vaultpw with the contents of the LastPass item named "password.carleton.edu Ansible vault pw". Set permissions with chmod go= .vaultpw.
1. ansible-vault create group_vars/cc_hosts/ansible_ssh_sudo_user.yml and add the following lines, substituting your actual username and password:

```
# file: inventory/group_vars/all/ansible_ssh_sudo_user.yml
# 
# FILE NOT INCLUDED IN GIT!!
# 
# Ansible ssh user and sudo ("become") user info.

vault_ansible_ssh_user: <<your_username>>
vault_ansible_become_password: "<<your_password>>"
```

# Deployment Instructions

## Installation
Self Service Password Reset (SSPR) is a web application. Currently, we are deploying SSPR as a WAR file running on a remote linux server with an Apache Tomcat web server installed. The WAR file contains an Apache Tomcat implementation of the SSPR version 4.6 application.

The user accounts SSPR manages are stored in a LDAP directory. The types of LDAP directories that SSPR supports are Active Directory, eDirectory, and Oracle Directory Server.

## Administrator Console
SSPR provides system administrators with an administrative console. Through the console, administrators can manage and configure all aspects of SSPR. Some important features include, importing and exporting the configuration files through the configuration manager, keeping track of the activity of the system through the administration dashboard, and editing the configuration through a UI in the configuration editor.

For further instructions and notes, refer to the official Microfocus documentation for SSPR at https://www.netiq.com/documentation/self-service-password-reset-46/

## Using this Repository
The configuration files for SSPR are managed in this github repository. All changes made to the production environment should be pushed to this directory. Any changes made in the control host can be reflected to the servers via ansible. Any changes made directly to the servers or via the configuration editor will be lost unless they are pushed to the github repository.

In order to reflect the changes you made to the servers running SSPR, you can run the playbook sspr-classic.yml in the main repository.
1. Make sure you are in the correct repository on your control host with the changes you have made. The file sspr-classic.yml should be in the current directory.
1. Run “ansible-playbook sspr-classic.yml" on the command line.
1. The playbook will execute and reflect any changes found in your repository to the files in the SSPR servers.

The SSPR Configuration file (SSPRConfiguration.xml) has credentials in it, and so to use it with version control we must first encrypt it. We are using ansible-vault to encrypt the file. If you make changes to the configuration of SSPR via the configuration editor follow the steps below to encrypt the file SSPRConfiguration.xml.
1. Download the configuration file from the configuration manager.
1. Copy the downloaded configuration file to roles/files/ directory
1. Encrypt with ansible-vault: ansible-vault encrypt roles/files/SSPRConfiguration.xml

## prod vs. dev vs. uat

The configuration of SSPR will be found in a file called SSPRConfiguration.xml. Currently we have the prod, dev, and uat environment. The dev environment will be where we will initially make any proposed changes to the configuration of SSPR. The prod environment will be the live version of SSPR available to all users. Before any changes made in dev are pushed to the prod environment, they should be user tested in the uat environment.

In order to make changes to a certain server, we can add the --limit  option when executing the ansible playbook. For example, to make changes only to the dev server, you could run the following in the command line: ```ansible-playbook --limit=password-dev.its.* sspr-classic.yml```


