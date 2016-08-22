# WSO2 Dashboard Server Puppet Module

This repository contains the Puppet Module for installing and configuring WSO2 Dashboard Server 2.0.0 on various environments. Configuration data is managed using [Hiera](http://docs.puppetlabs.com/hiera/1/). Hiera provides a mechanism for separating configuration data from Puppet scripts and managing them in a set of YAML files in a hierarchical manner.

## Supported Operating Systems

- Debian 6 or higher
- Ubuntu 12.04 or higher

## Supported Puppet Versions

- Puppet 2.7, 3 or newer

## Dependency Check

For puppet module to run it needs `ruby` pre-installed in the environment. Please check whether ruby is installed in the environment.

- ruby 1.9.3p484 x86_64-linux or above

## How to Contribute
Follow the steps mentioned in the [wiki](https://github.com/wso2/puppet-modules/wiki) to setup a development environment and update/create new puppet modules.

## Prepare the puppet module
WSO2 Base Puppet module is added as a submodule. Please run the following command to get the wso2base submodule.

        git submodule init
        git submodule update
        
## Packs to be Copied

Copy the following files to their corresponding locations.

1. WSO2 Dashboard Server distribution (2.1.0) to `<PUPPET_HOME>/modules/wso2ds/files`
2. JDK 1.7_80 distribution to `<PUPPET_HOME>/modules/wso2base/files`

Note:-
In default we have set the hostname of Dashboard Server as ds.dev.wso2.org in puppet module configuration files which can be found in 
`<PUPPET_HOME>/hieradata/dev/wso2/wso2ds/ 2.1.0/`. But in Dashboard Server 2.1.0 only contains certificate which is signed for localhost. 
So we need to import certificate signed for the hostname we configure to have and import them into `wso2carbon.jks` and `client-truststore.jks`  
and put these wso2carbon.jks and client-truststore.jks in the following location.  `<PUPPET_HOME>/modules/wso2ds/files/configs/repository/resources/security`.
(We have added the wso2carbon.jks and client-truststore.jks which has cert signed for default hostname ds.dev.wso2.org)

## Deploy WSO2 Dashboard Server as a Standalone
No changes to Hiera data are required to run the default profile.  Copy the above mentioned files to their corresponding locations and apply the Puppet Modules. 

But feel free to change the properties which available in the following location.  <PUPPET_HOME>/hieradata/dev/wso2/wso2ds/ 2.1.0/ 

1. If need to add mysql datasource instead of h2 please change the following in default.yaml. 

- First add Mysql connector for java (5.1.36) distribution to 
    <PUPPET_HOME>/modules/wso2ds/files/configs/repository/components/lib

- Uncomment the following line to add the mysql connector when deploying.
        
            wso2::file_list:
                -  repository/components/lib/%{hiera('wso2::datasources::mysql::connector_jar')}

- Uncomment and modify the MySQL based data sources to point to the external MySQL servers 

        EX:-
            wso2::master_datasources:
               wso2_gov_db:
                 name: WSO2_GOV_DB
                 description: The datasource used for config registry
                 driver_class_name: "%{hiera('wso2::datasources::mysql::driver_class_name')}"
                 url: jdbc:mysql://10.10.10.10:3306/DS_GOV_DB?autoReconnect=true
                 username: "%{hiera('wso2::datasources::mysql::username')}"
                 password: "%{hiera('wso2::datasources::mysql::password')}"
                 jndi_config: jdbc/WSO2_GOV_DB
                 max_active: "%{hiera('wso2::datasources::common::max_active')}"
                 max_wait: "%{hiera('wso2::datasources::common::max_wait')}"
                 test_on_borrow: "%{hiera('wso2::datasources::common::test_on_borrow')}"
                 default_auto_commit: "%{hiera('wso2::datasources::common::default_auto_commit')}"
                 validation_query: "%{hiera('wso2::datasources::mysql::validation_query')}"
                 validation_interval: "%{hiera('wso2::datasources::common::validation_interval')}"

- Make sure to uncomment “wso2::usermgt_datasource:” if adding a user management datasource. Please change the value to match the datasource name.

        EX:-
           wso2::usermgt_datasource:  wso2_user_db
     
2. (Option) Uncomment this to enable registry mounting. To apply this need a datasource for registry added under “wso2::master_datasources:”.

        EX:-
            wso2::registry_mounts:
                wso2_gov_db:
                    path: /_system/governance
                    target_path: /_system/governance
                    read_only: false
                    registry_root: /
                    enable_cache: true


3. If need to add proxy port for http/https please uncomment the following lines in default.yaml

        EX:-
            wso2::ports:
                proxyPort:
                    http: 80
                    https: 443


4. If need to change service provider details please change the details in “wso2::service_provider:” in default.yaml

        Ex:-
            wso2::service_provider:
                portal:
                    issuer: portal
                    acs:  https://ds.dev.wso2.org:9443/portal/acs


5. If need to change the identity provider details please change the details in “wso2::identity_provider:” in default.yaml.

        EX:-
            wso2::identity_provider:
                portal:
                    issuer: portal
                    identity_provider_url:  https://ds.dev.wso2.org:9443/samlsso


6. If need to change the authentication method please change the authentication method in “wso2::authentication:” in default.yaml

        EX:-
            wso2::authentication: sso


7. If need to change the authorization please change the details available under “wso2::authorization:” in default.yaml.

        EX:-
            wso2::authorization:
                activeMethod: oauth
                methods:
                    oauth:
                        idp_server: https://ds.dev.wso2.org:9443/oauth2/token
                        callback_url: https://ds.dev.wso2.org:9443/portal
                        client_name: portal
                        owner: admin
                        application_type: JaggeryApp
                        grant_type: password refresh_token urn:ietf:params:oauth:grant-type:saml2-bearer
                        saas_app: false
                        dynamic_client_registration_endpoint: https://ds.dev.wso2.org:9443/dynamic-client-web/register
                        token_scope: Production


8. To enable Dep sync please uncomment “wso2::dep_sync:”.

        EX:-
            wso2::dep_sync:
                enabled: false
                auto_checkout: true
                auto_commit: false
                repository_type: svn
                svn:
                    url: http://svnrepo.example.com/repos/
                    user: username
                    password: password
                    append_tenant_id: true



Note:-

No need to add new wso2carbon.jks and client-truststore.jks as mentioned in above section if you didn’t change the hostname available in 
default.yaml file. If you change the hostname you have to add wso2carbon.jks and client-truststore.jks which has cert for the changed hostname.

## Deploy WSO2 Dashboard Server as a Cluster

As there is only one profile available for WSO2 Dashboard Server deploying a cluster will be perform by appending separate Hiera data sets (yaml files) with 
clustering related configuration for WSO2 Dashboard Server for each node separated by server host name which are available under <PUPPET_HOME>/hieradata/dev/node. 
Yaml files’ names should match the server machine's’ host names.

There are couple sample files have added ( `wso2ds.2.1.0.dev1.wso2.org.yaml` and `wso2ds.2.1.0.dev2.wso2.org.yaml`) so only a few changes are needed to setup a
distributed deployment. For more details refer the WSO2 Dashboard Server clustering guide 

If went through the wso2ds.2.1.0.dev1.wso2.org.yaml and wso2ds.2.1.0.dev2.wso2.org.yaml files you can see the following configurations available which are 
need to be changed for each node.


1. If the Clustering Membership Scheme is `WKA`, add the Well Known Address list.
   Ex:
      
            wso2::clustering:
                   enabled: true
                   local_member_host: "%{::ipaddress}"
                   local_member_port: 4000
                   domain: ds.wso2.domain
                   subDomain: manager
                   membership_scheme: wka     
                   wka:
                       members:
                         -
                           hostname: 192.168.100.81
                           port: 4000
   
   
2. If dep sync is needed then make sure to enable it and add “wso2::dep_sync:” details per each node and make sure to add only one node to be both  auto_checkout and auto_commit enabled. 
   EX:-
   
       
         Node1:
          Wso2::dep_sync:
          enabled: false
          auto_checkout: true
          auto_commit: true
          repository_type: svn
          Svn:
             url: http://svnrepo.example.com/repos/
             user: username
             password: password
             append_tenant_id: true
   
       Node2:
        Wso2::dep_sync:
          enabled: false
          auto_checkout: true
          auto_commit: false
          repository_type: svn
          Svn:
            url: http://svnrepo.example.com/repos/
            user: username
            password: password
            append_tenant_id: true
       
   
   It is recommended to add external data sources when deploying as a cluster. Datasource configuration available under default.yaml available in  <PUPPET_HOME>/hieradata/dev/wso2/wso2ds/ 2.1.0/
   Configuring data source details inside the default.yaml would be enough as datasource should be shared for both nodes.
    
   Please make sure to change the details for “wso2::sso_authentication:”, “wso2::authorization:”, “wso2::identity_provider:” and “wso2::service_provider:” as per node.

## Running WSO2 Dashboard Server with Secure Vault

WSO2 Carbon products may contain sensitive information such as passwords in configuration files. WSO2 Securevault provides a solution for securing such information.

Uncomment and modify the below changes in Hiera file to apply Secure Vault.

1. Enable Secure Vault

        wso2::enable_secure_vault: true

2. Add Secure Vault configurations as below

        wso2::secure_vault_configs:
            <secure_vault_config_name>:
            secret_alias: <secret_alias>
            secret_alias_value: <secret_alias_value>
            password: <password>

        Ex:-
            wso2::secure_vault_configs:
                key_store_password:
                secret_alias: Carbon.Security.KeyStore.Password
                secret_alias_value: repository/conf/carbon.xml//Server/Security/KeyStore/Password,false
                password: wso2carbon

3. Add Cipher Tool configuration file templates to `template_list`

        EX:-
            wso2::template_list:
                - repository/conf/security/cipher-text.properties
                - repository/conf/security/cipher-tool.properties
                - bin/ciphertool.sh

## Running WSO2 Dashboard Server on Kubernetes

WSO2 Dashboard Server Puppet Module ships Hiera data required to deploy WSO2 Dashboard Server on Kubernetes. For more information refer to
the documentation on [deploying WSO2 products on Kubernetes](https://docs.wso2.com/display/PM210/Deploying+WSO2+Products+on+Kubernetes+Using+WSO2+Puppet+Modules) using WSO2 Puppet Modules.

## Reference
https://github.com/wso2/puppet-modules/wiki
