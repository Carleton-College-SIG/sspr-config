# Deployment Instructions for Certbot

## Introduction
Sectigo Certificate Manager supports the Automatic Certificate Management Environment (ACME) protocol, which automates interactions between a CA and Carleton’s web servers, including certificate installation, renewal, and domain validation. Using a third-party ACME client (Certbot), customers can connect to Sectigo ACME servers (the endpoints) via an appropriately configured ACME endpoint account. The ACME endpoint can then be used by the ACME client to enroll and manage SSL certificates.

## Environment
The deployment includes configuring Certbot on the following servers:
### 1. SSPR Servers:
- Operating System: Rocky 8/9 Linux
- Web Server: Apache Tomcat
- Playbook: sspr-classic.yml
## Deployment Steps
### 1. Prerequisites
- Ensure that servers are running the correct operating system mentioned above
- Ensure that ansible playbooks are setup for the environment
### 2. ACME Setup
Managing ACME end points
- a. In SCM, from the top left drop down menu, go to Enrollment -> ACME
- b. You will see a list of ACME endpoints
- c. You will be using the endpoint with ECCOV. Select the endpoint and view the Accounts associated with it.
- d. When creating account make sure domains \*.carleton.edu and carleton.edu are both working
- e. Once the account is created, you can check the external account binding (eab) values by clicking on the “details” for each account. These values are used to connect with the individual servers for certificates.
### 3. Certbot Installation
The certbot role (`roles/certbot/tasks/main.yml`) will take the following steps necessary to install certbot and obtain certificates.
- a. Download and configure the appropriate packages to run certbot (epel-release, snapd)
- b. Install certbot using snapd
- c. Execute certbot to obtain a certificate if one does not already exist  at `/etc/letsencrypt/`
- d. Change permissions on the certificate key files
- e. Create cron job for certificate renewal

To ensure secure communication, Carleton servers are located behind a load balancer,
not visible from the external environment. As a result, we have chosen to use the
--manual plugin, instead of the standard --nginx and --apache authenticator plugins for
certbot in our environment, by utilizing the external account binding values of the ACME
endpoints. The necessary authentication details are stored securely within the `cli.ini`
configuration file.

Before running the playbook make sure of the following
- The eab keys for the certificate generation is stored in the `cli.ini` file in “`roles/certbot/templates/cli.ini.j2`”. The file is ansible encrypted. Make sure that the eab values stored are correct in the configuration file.
- Check the file `staging` and make sure the target host is listed under `certbot_hosts` and any other appropriate host groups.
### 4.Certificate Renewal
Due to the use of the `--manual` plugin with the provided EAB (External Account Binding) keys from InCommon, the standard certbot renew command cannot be used for automatic certificate renewals. Instead, we need to re-run the original certbot command to renew the certificate.

To achieve automated certificate renewal, we create a daily cron task that runs the certbot command. This ensures that the certificate is checked and renewed if necessary on a daily basis.

**Note:** We include the `--keep-until-expiring` option in the original renewal code to maintain the non-interactive nature of the playbook. Without this option, when Certbot finds an existing non-expired certificate, it prompts the user to choose whether or not to renew the certificate. By utilizing the `--keep-until-expiring` option, we ensure that the renewal process remains fully automated and does not require manual intervention.
### 5.cleanup.yml
We have included a cleanup playbook, cleanup.yml (or cleanup-idp/lb.yml) which can be used to reverse or clean up the changes made by `idp.yml`,`lb.yml`, and `sspr-classic.yml` playbooks. The purpose of
this playbook is to prepare a clean start environment in case of any software updates or when a reset of the configuration is required.

**Important:** Please note that running the cleanup playbooks will revoke and delete the
existing SSL certificate located at `/etc/letsencrypt/`. Exercise caution when executing this playbook to avoid unintended certificate revocation.


