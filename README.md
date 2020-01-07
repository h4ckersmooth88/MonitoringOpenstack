# Monitoring Services and Logging for RedHat Openstack (RHOSP 12-15)



## CHAPTER 1. Introduction

Monitoring tools are an optional suite of tools designed to help  operators maintain an OpenStack environment. We can use tools : 	

- Centralized logging: Allows you gather logs from all components in  the OpenStack environment in one central location. You can identify  problems across all nodes and services, and optionally, export the log  data to Red Hat for assistance in diagnosing problems.

  

  ![](https://access.redhat.com/webassets/avalon/d/Red_Hat_OpenStack_Platform-15-Monitoring_Tools_Configuration_Guide-en-US/images/bd2b529a7165f6ca66255e1edef13cba/centralised_logging_single_node_fluentd.png)		

  The centralized logging toolchain consists of a number of components, including: 		

  - Log Collection Agent (Fluentd) 				
  - Log Relay/Transformer (Fluentd) 				
  - Data Store (Elasticsearch) 				
  - API/Presentation Layer (Kibana) 				



- Availability monitoring: Allows you to monitor all components in  the OpenStack environment and determine if any components are currently  experiencing outages or are otherwise not functional. You can also  configure the system to alert you when problems are identified. 			

  ![](https://access.redhat.com/webassets/avalon/d/Red_Hat_OpenStack_Platform-15-Monitoring_Tools_Configuration_Guide-en-US/images/f1846718cb48b62661afc70ce8538b7b/availability_monitoring_single_node_sensu.png)The availability monitoring toolchain consists of a number of components, including: 		

  -  Monitoring Agent (Sensu client) 				
  -  Monitoring Relay/Proxy (RabbitMQ) 				
  -  Monitoring Controller/Server (Sensu server) 				
  -  API/Presentation Layer (Uchiwa) 				



#### The conclusion, we use:

> Fluentd – open source data collector for logging
>
> Integrates with: Elasticsearch, Kibana
>
> Sensu - Monitor servers, services, application health
>
> Integrates with: Uchiwa

for the Monitoring Resource and Metric, I will create in another Tutorial



## CHAPTER 2. Install the Server-Side Tools

Please prepare One VM Centos 7 OS for Server,

Please not create VM in openstack environment, the OpsTools Server must install in external environment

```
[root@opstools ~]# yum install git ansible 

[root@opstools ~]# git clone https://github.com/centos-opstools/opstools-ansible.git 

[root@opstools ~]# cd opstools-ansible/ 

[root@opstools opstools-ansible]# ssh-copy-id root@localhost
```

Now we must define hosts inventory file

```
[root@opstools opstools-ansible]# vim inventory/hosts
opstools ansible_host=localhost ansible_user=root ansible_become=true

[am_hosts]
opstools

[logging_hosts]
opstools
```

And we create config.yml to define username and password

```
 [root@opstools opstools-ansible]# vi config.yml
 uchiwa_credentials:
   - username: 'uchiwa'
     password: 'changeme'
 kibana_credentials:
   - username: 'kibana'
   password: 'changeme' 
```

and Install all the dashboards with a single playbook :

```
[root@opstools opstools-ansible]# ansible-playbook playbook.yml -e @config.yml

```

Please make sure status installation is ok

The successfully deployed opstools server will result in following message:

```
PLAY RECAP  *****************************************************************************************************************************************************************************************************************************************************************************************
 opstools          : ok=187 changed=39  unreachable=0  failed=0   
```



## CHAPTER 3. Install the Client-Side Tools

Next copy default opstools configuration yaml files to you local templates directory:

```
(undercloud) [stack@undercloud ~]$ cp /usr/share/openstack-tripleo-heat-templates/environments/logging-environment.yaml templates/

(undercloud) [stack@undercloud ~]$ cp  /usr/share/openstack-tripleo-heat-templates/environments/monitoring-environment.yaml templates/

```

Now we definition IP Server Opstools in YAML

```
(undercloud) [stack@undercloud templates]$ vi logging-environment.yaml
## A Heat environment file which can be used to set up
## logging agents

resource_registry:
  OS::TripleO::Services::FluentdClient: /usr/share/openstack-tripleo-heat-templates/docker/services/fluentd.yaml

parameter_defaults:

## Simple configuration
#
LoggingServers:
 - host: 10.10.10.1
   port: 24224

#   - host: log1.example.com
#     port: 24224
#
## Example SSL configuration
## (note the use of port 24284 for ssl connections)
#
# LoggingServers:
#   - host: 192.168.24.11

#     port: 24284
# LoggingUsesSSL: true
# LoggingSharedKey: secret
# LoggingSSLCertificate: |
#   -----BEGIN CERTIFICATE-----
#   ...certificate data here...
#   -----END CERTIFICATE-----
```



```
(undercloud) [stack@undercloud templates]$ vi monitoring-environment.yaml
## A Heat environment file which can be used to set up monitoring agents

resource_registry:
  OS::TripleO::Services::SensuClient: /usr/share/openstack-tripleo-heat-templates/docker/services/sensu-client.yaml

parameter_defaults:
  MonitoringRabbitHost: 10.10.10.1

  MonitoringRabbitPort: 5672
  MonitoringRabbitUserName: sensu
  MonitoringRabbitPassword: sensu
#  MonitoringRabbitUseSSL: false
#  MonitoringRabbitVhost: "/sensu"
#  SensuClientCustomConfig:
#    api:
#      warning: 10
#      critical: 20


```

Finally we deploy in your Overcloud

Include the modified YAML files with your `openstack overcloud deploy` command to install the Sensu client and Fluentd tools on all overcloud nodes. For example: 

```
(undercloud) [stack@undercloud templates]$ openstack overcloud deploy --templates  -e /usr/share/openstack-tripleo-heat-templates/environments/network-isolation.yaml -e network-environment.yaml -e ~/templates/monitoring-environment.yaml -e ~/templates/logging-environment.yaml --control-scale 3 --compute-scale 1 --ntp-server 192.168.122.10
```

you must define in your environment :

1. ~/templates/monitoring-environment.yaml
2. ~/templates/logging-environment.yaml



## CHAPTER 4. Access the Monitoring Tools

You can access in here :

 https://<opstools-ip-or-host>/kibana

![](https://raw.githubusercontent.com/h4ckersmooth88/MonitoringOpenstack/master/Kibana.png)



https://<opstools-ip-or-host>/uchiwa 

![](https://raw.githubusercontent.com/h4ckersmooth88/MonitoringOpenstack/master/uciwa.png)