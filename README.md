## Table of contents
* [Description](#Description)
* [Requirements](#Requirements])
* [Procedure](#Procedure)
* [Variables](#Variables)
* [Dependencies](#Dependencies)
* [Author Information](#Author)



# Description
EDIGW CD covering installation of specific components in scope, further deployment for the same components, DB update, Rollback and Test Application

# Requirements
* Standard Ansible prerequisities of WinRM configuration in managed nodes
* Presence of targeted WebServer, Batch and SQL Server host in Ansible inventory
* Presence of host variable **_ostype_** for managed nodes to indicate whether its __windows__ or __not__

# Procedure

## Create Job Template - CAPS

* Name: gwt_jobtemplate_edigwdps_deploy_standalone
* Description:  This play helps to deploy EDIGW 277 CA processor service
* Job Type: Run
* Inventory: gwt_inventory
* Project: gwt_project_edigw (Project synced to [this repo](https://github.com/mygainwell/pfe-oc-edigw-deploy))
* Playbook: **deploy.yml**
* Credentials: gwt_cred_ansible_vault, gwt_cred_jfrog, gwt_cred_msssql, gwt_cred_windows
* Variables: Prompt on Launch
* Limit: Prompt on Launch
* Enable Concurrent Jobs: True
* Job Tags: caps

```diff
- While applying limit when the job template is launched, make sure to include only applicable Servers specific to DPS component
```

## Create Job Template - DB

* Name: gwt_jobtemplate_edigwdb_deploy_standalone
* Description:  This play helps to deploy EDI_Gateway Database
* Job Type: Run
* Inventory: gwt_inventory
* Project: gwt_project_edigw (Project synced to [this repo](https://github.com/mygainwell/pfe-oc-edigw-deploy))
* Playbook: **deploy.yml**
* Credentials: gwt_cred_ansible_vault, gwt_cred_jfrog, gwt_cred_msssql, gwt_cred_windows
* Variables: Prompt on Launch
* Limit: Prompt on Launch
* Enable Concurrent Jobs: True
* Job Tags: db

```diff
- While applying limit when the job template is launched, make sure to include only applicable Servers specific to db component
```

# Variables

## Survey Variable - CD (Optional)
Name | Type | Choices | Default | Required | Question | Description
--------|---------|--------|-------|-------|----------|----------
__jfrog_artifact__ | Text |  |  | No | Relative Path of Artifact in JFrog | Artifact Compressed File Location relative to JFrog Base URL.
__is_rollback__ | Multiple Choice (single select) | true<br>false | false | No | Do you want to initiate Rollback? | If this option is chosen as true, application will be redeployed with previous package specified in install history and DB will be restored from backup taken before last successful deployment (rollback will not be considered as a deployment).

## Extra Variable - CAPS
  Update the variable values accordingly to meet the need of Product deployment

    #Name of the Product
    product_name: "EDIGW-CAPS"
    #Name of the Product Folder Name in Installation Location
    product_install_name: "EDIGW 277CA Processor service"
    #Name of the Product Config File
    product_install_config: "EDIGW_277CA_Processor_Service.exe.config"
    #Product Component Short Name
    product_install_type: "caps"
    #Staging Path in target node where artifact has to be downloaded and processed
    artifact_temp: 'H:\Ansible\CD'
    #Path of the location to keep Backups
    uninstall_backup_dest: 'H:\Ansible\CD\Backup'

  Incase survey is not enabled, provide value for below variables while launching for deployment

    #Relative Path of Artifact in JFrog
    jfrog_artifact:

  Incase of rollback, set below variable

    is_rollback: true

## Extra Variable - DB
  Update the variable values accordingly to meet the need of Product deployment

    #Name of the Product
    product_name: "EDIGW-DB"
    #Name of the Product Folder Name in Installation Location
    product_install_name: "EDI Gateway DataExtract"
    #Name of the Product Config File
    product_install_config: "HPP.EDIGateway.DataExtract.exe.config"
    #Product Component Short Name
    product_install_type: "db"
    #Name of the ConnectionString in Application config file to identify DB information
    product_dbconnstring_name: "DBConnectionString"
    #Staging Path in target node where artifact has to be downloaded and processed
    artifact_temp: 'H:\Ansible\CD'
    #Path of the location to keep Backups
    uninstall_backup_dest: 'H:\Ansible\CD\Backup'

  Incase survey is not enabled, provide value for below variables while launching for deployment

    #Relative Path of Artifact in JFrog
    jfrog_artifact:

  Incase of rollback, set below variable

    is_rollback: true

# Dependencies
This playbook needs to be run from Ansible Tower applicable to the organization.

When configuring template from a project using this playbook, you need to have the following credentials:

    Credential of type Machine to access the managed node with admin privilege
    Credential of type Database to access SQL instance with admin privilege
    Credential of type Artifactory to download artifact from JFrog
    Credential of type Vault to decrypt the passwords assigned to encrypted variable

Following roles are utilized by this playbook:

    pfe-oc-cd-getartifact
    pfe-oc-cd-extractartifact
    cd-edigw-installcaps
    cd-edigw-rollback
    cd-edigw-testapp
    cd-edigw-updatedb

```diff
- It is assumed that managed nodes are reachable from Ansible Tower without any additional hops.
```

# Author
Contribution/Updates: Vijayakumar Sankaramoorthy (v.sankaramoorthy@gainwelltechnologies.com)
