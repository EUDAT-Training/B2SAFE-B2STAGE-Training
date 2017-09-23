# B2SAFE-B2STAGE-Training
<img align="right" src="img/B2HANDLE.png" width="100px">
<img align="right" src="img/B2STAGE.png" width="100px">
<img align="right" src="img/B2SAFE.png" width="100px">

## Contents
This training module provides hands-on material for iRODS4, EUDAT B2SAFE, B2STAGE and persistent identifiers, more specifically handles (handle version 8).

Please consult the user [documentation on the services](https://eudat.eu/services/userdoc) for a general introduction, if needed, before following the contents of this git repository. This training material foresees two types of trainees: those who want to learn how to **use** the EUDAT B2SAFE and B2STAGE services; and those who prefer to **deploy** and integrate these services. Following the full, in-depth tutorial will allow you to understand how the components of a service are combined and thus enables you to also extend the integration of services at the low-level (technology-level rather than API level). Following just the "use" part of the training will familiarise you with the APIs of the services, but not with the underlying technology and its wiring. 

The order of the markdown files proposes the curriculum of the training. Each component takes about 1 hour. 

File | Target audience | Status
------|--------------|-----
<span class="css-truncate css-truncate-target"><a href="/00-install-iRODS4.md" class="js-navigation-open" title="00-install-iRODS4.md">00-install-iRODS4.md</a></span> | site admins
<span class="css-truncate css-truncate-target"><a href="/01-iRODS-handson-user.md" class="js-navigation-open" title="01-iRODS-handson-user.md">01-iRODS-handson-user.md</a></span> | researchers
<span class="css-truncate css-truncate-target"><a href="/02-iRODS-handson-admin.md" class="js-navigation-open" title="02-iRODS-handson-admin.md">02-iRODS-handson-admin.md</a></span>	| site admins and interested researchers
<span class="css-truncate css-truncate-target"><a href="/03-install-B2SAFE.md" class="js-navigation-open" title="03-install-B2SAFE.md">03-install-B2SAFE.md</a></span>	| site admins
<span class="css-truncate css-truncate-target"><a href="/04-iRODS_federations_configuration.md" class="js-navigation-open"  title="04-iRODS_federations_configuration.md">04-iRODS_federations_configuration.md</a></span>	| site admins and interested researchers
<span class="css-truncate css-truncate-target"><a href="/05-iRODS-advanced-users.md" class="js-navigation-open" title="05-iRODS-advanced-users.md">05-iRODS-advanced-users.md</a></span>| site admins and researchers
<span class="css-truncate css-truncate-target"><a href="/06-B2SAFE-handson.md" class="js-navigation-open" title="06-B2SAFE-handson.md">06-B2SAFE-handson.md</a></span>| site admins and data managers
<span class="css-truncate css-truncate-target"><a href="/07a-Working-with-PIDs_CURL.md" class="js-navigation-open" title="07a-Working-with-PIDs_CURL.md">07a-Working-with-PIDs_CURL.md</a></span> <p><span class="css-truncate css-truncate-target"><a class="js-navigation-open" title="07b-Working-with-PIDs_epicclient.md">07b-Working-with-PIDs_epicclient.md</a></span> <p><span class="css-truncate css-truncate-target"><a href="/07c-Working-with-PIDs_B2HANDLE.md" class="js-navigation-open" title="07c-Working-with-PIDs_B2HANDLE.md">07c-Working-with-PIDs_B2HANDLE.md</a></span> <p><span class="css-truncate css-truncate-target"><a class="js-navigation-open" title="07d-Working-with-PIDs_EPIC.md">07d-Working-with-PIDs_EPIC.md</a></span> | site admins and interested researchers | [Legacy material] Module 07b and Module 07d \*
<span class="css-truncate css-truncate-target"><a href="/08-install-gridFTP-server.md" class="js-navigation-open" title="08-install-gridFTP-server.md">08-install-gridFTP-server.md</a></span> | site admins
<span class="css-truncate css-truncate-target"><a href="/09-install-B2STAGE.md" class="js-navigation-open" title="09-install-B2STAGE.md">09-install-B2STAGE.md</a></span> | site admins
<span class="css-truncate css-truncate-target"><a href="/10-using-B2STAGE.md" class="js-navigation-open" title="10-using-B2STAGE.md">10-using-B2STAGE.md</a></span> | researchers, last exercise is a joint effort between researchers and site admins

## Accesss to the EUDAT Training Testbed
To get access to the training environment, please use the [EUDAT contact pages](https://eudat.eu/support-request?service=DOCUMENTATION); and provide some details on which community you are from and in which context you would like to follow the tutorial. 

## Set up
During the tutorial site admins learn how to set up an infrastructure like indicated in the picture below. Users can make use of a pre-deployed instance of this infratructure on the EUDAT Training Sandbox. 

We provide a detailed tutorial for **iRODS 4.1.10 on Ubuntu 14** in the main modules. However, since Ubuntu 14 is out of support but there are no official packages for Ubuntu 16 yet, we provide details on the setup below for **CentOS 7.X and iRODS 4.2.1** in [TrainingInstances-iRODS4.2.1](TrainingInstances-iRODS4.2.1). In that case please replace all modules for installations with the instructions provided there.

<img align="centre" src="img/VM-setup.png" width="800px">

### Users - Training
Users can get access to a setup of virtual machines (VMs) like above. The tutorial will show them the functionality of single components and how to combine them in order to arrive at proper data management. Users can either choose to set up their personal computer to resemble the user interface machine or they can receive a login on a user interface VM on the training sandbox operated by the EUDAT User Documentation and Training Material team. Via the user interface machine they can access the first VM which contains an iRODS server, a gridFTP server, and the B2SAFE module. Another VM holds another iRODS server. Both iRODS grids are federtated and users can transfer data from one grid to the other. The user interface VM also provides the necessary python libraries to work with the *epicclient* and with B2HANDLE.

The appropriate parts of the tutorial you can follow in the training environment are labeled with "researchers", "interested researchers" and "data managers" in the table above.

### Site admins - Training
Site admins will be guided through all steps to set up the environment, covering iRODS installation and federation, setting up B2SAFE, deploying a gridFTP endpoint and pointing it to the iRODS server via the data staging interface. 

To build the setup you will need to prepare or have access to three distinct (potentially virtualised) computational resources; each with e.g. 2 vCPU, 8GB memory, 100GB disk; running a Linux operating system (Ubuntu preferred); in which you have sudo rights. 

You will also need extra credentials for the persistent identifiers part; see below. 

### Persistent identifiers (PIDs)
In the tutorial we  explain how to create, update and delete PIDs. The infrastructure to manage PIDs is provided by SURFsara. Note that all instances (B2SAFE, user modules) will make use of the same PID prefix. 

To follow this part of the tutorial you need extra credentials. We will provide you with these credentials via [EUDAT contact pages](https://eudat.eu/support-request?service=DOCUMENTATION); please provide some details on which community you are from and in which context you would like to follow the tutorial. 

## Legacy material
\* To get back to the old versions do:
```sh
git clone https://github.com/EUDAT-Training/B2SAFE-B2STAGE-Training.git
git checkout 7b22697f13efce29465d97dfb3177be5db2e33dc 07d-Working-with-PIDs_EPIC.md
git checkout 7b22697f13efce29465d97dfb3177be5db2e33dc 07b-Working-with-PIDs_epicclient.md
```
