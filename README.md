# sspr-config
Carleton SSPR configuration

## Using this repository

The SSPR configuration has credentials in it, and so to use it with version control we must first encrypt it.  We are using ansible-vault to encrypt the file, although we are not yet using Ansible for any other aspect of configuration.

1. The first step is to clone this repository, then grab the password from LastPass (stored as "password.carleton.edu Ansible vault pw") and save it in the cloned repository as ".vaultpw".  (This is the same place it is stored in a typical Carleton Ansible configuration repository.)
1. Update the SSPR configuration within the SSPR application:

https://password.carleton.edu/sspr/private/config/editor

1. After the changes are saved in the application, download the configuration file from the configuration manager:

https://password.carleton.edu/sspr/private/config/manager

1. Copy the downloaded file into the files/ directory within the configuratuin.

1. Encrypt with ansible-vault

> ansible-vault encrypt files/SSPRConfiguration.xml

To restore a previous version of the configuration:

1. Download the current configuration, in case you need to refer to it.
1. Copy the list of domain controllers being used, as these occasionally change.  (You can also get it from the configuration copy you just downloaded.)
1. Check out the repository at the commit you want to recover.
1. Decrypt the configuration with ansible-vault:

> ansible-vault decrypt files/SSPRConfiguration.xml

1. Upload the configuration using the appliance's configuration manager:

https://password.carleton.edu/sspr/private/config/manager

1. Check the list of domain controllers being used.  Download the current certs from the domain controllers.
1. Re-encrypt the file in the repository, or restore the encrypted version within git.  See "git status" for the exact commands.

## Production vs. dev

The production configuration is in files/SSPRConfiguration.xml.  At the time of this writing, there are no development or test instances.  Documentation for those configurations should be created when those servers are established. 
