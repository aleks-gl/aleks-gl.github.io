# Apache Airflow Installation Concept
## Introduction
This manual contains common description of Apache Airflow installation process for Linux based operating system. 
Presupposes that those interested in this document are familiar with the Apache Airflow tool, however, 
we will give a brief reference about the purpose and capabilities of this software.

### Brief reference
_Apache Airflow_ — it is platform for programmatically manage of workflows, in other words – for run, 
schedule and control sequences of processes. This tool is good solution for several reasons. 
Apache Airflow is user oriented such as it has GUI for run and control workflows. 
Workflow in Airflow is presented as as Directed Acyclic Graphs. This means that we have well formed and controlled 
sequence of tasks. In other side, Airflow can be good solution such as it is can use different type of operations 
for task in workflow. For example, bash script or python code can be used as task. This product is open source project 
of Apache Software Foundation what provides an opportunity for improvement to suit your needs.  
More information about this tool can founded in documentation for Airflow.  
___Purpose of this document___ is define recommendation for deployment Apache Airflow on Linux based systems 
(prefer for Ubuntu and RedHat Linux).  

## Preparation
Number of using conditions must be refined to create precise description of deployment task. 
Before starting deploy we should have explicit definitions for:
1. application area of the product and target environment for product application must be defined
2. system architecture
3. security politics and rules. 

All of this points must be defined as architect concept of system and now we expose only relations between above 
listed points and configuration steps.

### Application area and target environment
Listed below configurations is depending of the application area.
1. ___Apache Airflow is used in testing area or for development.___ In this case system have no high load but frequent 
   configuration changes occur and the system requires the ability to restore the original state. 
   In addition, the system must be able to install additional modules (extra). In this case, the set of stored 
   technological data is not important and it is possible to use SQLite database and default settings  for Apache Airflow. 
   Basic configuration parameters, such as the path to the repository with tasks, are changed by scripts 
   for deploying the application.
2. ___Apache Airflow is used in productive space.___ In this case, the system is subjected to consistently high loads 
   and rare changes. In addition, it is assumed that the system must have a fault-tolerant configuration. 
   If the system is used in the productive area, we must use a special assembly for the Docker image and PostgreSQL DBMS.  

###   System architecture
The target infrastructure configuration accordingly determines how the Apache Airflow platform is deployed. 
For the purposes of this tutorial, we assume the deployment of the Apache Airflow software in the standard versions 
offered in the documentation for this product:
1. Installing Apache Airflow as standalone instance or named ___local installation___. 
   This version of deployment can be used both for installation on a physical dedicated server and for a 
   Linux server running on a virtual machine.
2. ___Installing Apache Airflow as Docker image.___ In this case we have scalable system with other advantages of 
   virtualization. Also this version of deployment can used in many types of computers such as dedicated servers 
   and cloud architecture.  
   
###   Security politics and rules
Because security configuration should be carried out on the precise recommendations built on  company's security policy,
so in this manual, we will clarify only the main aspects that should be important to deployment process. 
Apache Airflow documentation contain precise description of user roles, that used by default. In this manual we suppose 
that default roles is used for create conception of security configuration. Function for create specific roles for 
adjust security also available in Apache Airflow. 
   At this point we should define roles and access rights for them. As example we can define next roles:  
1. ___Admin___ – users, that have access to edit configurations and manage tasks
2. ___Users___ – some peoples that have acess rights for start or stop tasks and view process status
3. ___Viewer___ – those role means only visual control of process status
   and we can limit all access rights for others (___Public___).  
   
_Practical tips:_  
 Entry point for define security configuration is file *airflow.cfg* in home directory.  
 Specification of access rules is defined in *airflow/www/security.py*. 
### Package specification
   _Recommendations for Apache Airflow environment:_  
   + Python: 3.6, 3.7, 3.8  
   Databases:
   + PostgreSQL: 9.6, 10, 11, 12, 13
   + MySQL: 5.7, 8
   + SQLite: 3.15.0+
   Kubernetes: 1.16.9, 1.17.5, 1.18.6  
     
   For the purposes of this tutorial we approve next configurations:  
   _DBMS for Airflow:_  
   + PostgreSQL 11.* must used in productive area
   + SQLite can used in area for development and tests.  
   _kubernetes_ — we can use any of the versions recommended for Apache Airflow and available 
     within the target infrastructure.  
     
   MySQL DBMS version is not implied, both due to limitations for creating parallel executors and limitations
     that will allowed in process of creating a custom image for Docker.
   ### Apache Airflow deployment types
   In result of specifications processing in previous points we can define next common types for Apache Airflow deployment:  
   + local installation of Apache Airflow for a test environment (or development environment) with SQLite DBMS;
   + installing Apache Airflow as a Docker image for a test environment (or development environment) with SQLite DBMS;
   + local installation Apache Airflow for a test environment (or development environment) with PostgreSQL DBMS;
   + installing Apache Airflow as a Docker image for a test environment (or development environment) with PostgreSQL DBMS;
   + local installation Apache Airflow for a productive environment with PostgreSQL DBMS;
   + installing Apache Airflow as a Docker image for a productive environment with PostgreSQL DBMS

For each of listed variants we have also additional division to::  
   + Apache Airflow will run as dedicated server for workflow management;
   + Apache Airflow will run in a cluster.  

Adjustment for both of configuration types for use as part of a distributed system or as a dedicated server should stored 
in configuration files and it should be fixed in installation scripts.  

_Practical tips:_  
Entry point for define configuration it is file airflow.cfg in home directory.  

## Deployment Apache Airflow
### Basic sequence of actions
Apache Airflow installation process for all deployment styles should contain standard steps like installation process 
for every software in Linux systems. So, process of installation should include:
1. check for exists needed software and libraries for run Apache Airflow at the target node
2. install Apache Airflow
3. initialize database
4. final adjustment of application.  
Each of these points may have additional steps, but we are only specifying the basic steps.  

### Install software environment for Apache Airflow
To reduce the number of steps, the installation of the software environment libraries is combined into one command,
without division into functional groups. In case of installation on a production system, it is recommended to determine
the list of required libraries and install only the required libraries.  
In the first step, install the basic set of libraries required for Apache Airflow:  

_sudo apt-get install -y --no-install-recommends \
freetds-bin krb5-user ldap-utils libffi6 libsasl2-2 libsasl2-modules libssl1.1 locales  lsb-release sasl2-bin unixodbc 
python3.6-dev openssl sqlite sqlite-dev default-libmysqlclient-dev libmysqld-dev postgresql_

This is example for Ubuntu server. Accordingly, other Linux distributions use their own package managers, for example, 
for RedHat, Fedora and CentOS we need to use the _yum_ package manager, for SUSE we should use _zypper_ software.  

### Apache Airflow installation.
#### Plain installation
If we don not needed for a specialized version that have fixes of source code or that have version of the application 
that differs from the one supplied by default, a quick installation is possible using pip for the dedicated server option 
or docker to get an image.  
For _local install_ of Apache Airflow will run next Bash script:  

_AIRFLOW_VERSION=2.0.1
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
EXTRAS=”postgres,google”
pip install "apache-airflow[${EXTRAS}]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"_  

So, variable _EXTRAS_ will contain list of additional packages, that listed below.

___Additional packages:___  
airbyte, all, all_dbs, amazon, apache.atlas, apache.beam, apache.cassandra, apache.druid,apache.hdfs, apache.hive, 
apache.kylin, apache.livy, apache.pig, apache.pinot, apache.spark,apache.sqoop, apache.webhdfs, async, atlas, aws, azure, 
cassandra, celery, cgroups, cloudant,cncf.kubernetes, crypto, dask, databricks, datadog, devel, devel_all, devel_ci, 
devel_hadoop,dingding, discord, doc, docker, druid, elasticsearch, exasol, facebook, ftp, gcp, gcp_api,github_enterprise, 
google, google_auth, grpc, hashicorp, hdfs, hive, http, imap, jdbc, jenkins,jira, kerberos, kubernetes, ldap, 
microsoft.azure, microsoft.mssql, microsoft.winrm, mongo, mssql,mysql, neo4j, odbc, openfaas, opsgenie, oracle, pagerduty, 
papermill, password, pinot, plexus,postgres, presto, qds, qubole, rabbitmq, redis, s3, salesforce, samba, segment, sendgrid, 
sentry,sftp, singularity, slack, snowflake, spark, sqlite, ssh, statsd, tableau, telegram, vertica,virtualenv, webhdfs, 
winrm, yandex, zendesk.  

To install all available packages, you can set the parameter EXTRAS=”all”. Accordingly, for a production environment, 
it is not recommended to install all packages and it is necessary to list the list of packages that will be used.  
#### Plain install of Docket image
To get a Docker image with latest default version of Apache Airflow from a public register, run the command:  

_docker pull apache/airflow_  

#### Special version installation
The following steps describe how to get the Apache Airflow source code from the https://github.com/apache/airflow.git 
repository and build and install it. The following method involves more steps, but it has a number of advantages. 
When installing from the source code, we can select a version (switch to git branch) and / or modify the source code 
if necessary.   
At first Apache Airflow source code should downloaded from the repository:  

_git clone https://github.com/apache/airflow.git_  

If there is no need to install additional packages them just use pip to install the downloaded package:  
_pip install ._  
or  
_pip setup.py install_  

If additional packages is needed for installation, list them in square brackets:  
_pip install .[async,google,amazon] --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-master/constraints-3.6.txt"_

#### Build specialized version of Docker image
Building an image for CI can be done using the breeze command that included in Apache Airflow source package. 
From root of the source code directory run the command:  

_./breeze build-image_  

In order to build an image for use in a production environment, version of the assembly should clarified:  

_./breeze build-image —production-image_  

Accordingly, for building an image that include additional packages list of them must be specified in parameter extras:  

_./breeze build-image --python 3.7 --extras "all" --production-image_  

List of additional packages is presented above.

### Initialization of Apache Airflow
#### Database initialization.
Before starting the Apache Airflow software, data should be initialized as briefly described below. 
Also before performing the initial data initialization, configuration for connection to the data source should be established, directories 
must be created and user rights assigned to these directories.  
In the case of a local installation, we execute the script to initialize data and launch application:  

_#airflow needs a home, ~/airflow is the default,_  
_#but you can lay foundation somewhere else if you prefer_  
_#(optional)_  
export AIRFLOW_HOME=~/airflow  

AIRFLOW_VERSION=2.0.1  
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"  
_# For example: 3.6_  
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"  
_# For example: https://raw.githubusercontent.com/apache/airflow/constraints-2.0.1/constraints-3.6.txt_  
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"  

_# initialize the database_  
airflow db init  

airflow users create --username admin --firstname Peter --lastname Parker --role Admin --email spiderman@superhero.org   
  
_# start the web server, default port is 8080_  
airflow webserver --port 8080  

_# start the scheduler_  
_# open a new terminal or else run webserver with ``-D`` option to run it as a daemon_  
airflow scheduler  

_# visit localhost:8080 in the browser and use the admin account you just_  
_# created to login. Enable the example_bash_operator dag in the home page_  

In the case of start ___Apache Airflow as Docker image___, first create the necessary directories in the home folder: 

_mkdir ./dags ./logs ./plugins_  
_echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env_  

On the system with the Apache Airflow image installed, run the command to start the data initialization procedure.  

_docker-compose up airflow-init_

After completing the data initialization procedure, services can started.  

_docker-compose up_  

## Final configuration of application
The final configuration of Apache Airflow implies performing actions that, for example, should provide:  
+ logging and monitoring of application
+ create of connections for interaction with served or processed systems
+ writing code to define workflows
+ procedures for integration into infrastructure, such as API integration for control and messaging.  

_Practical tips:_  
Entry point for define configuration it is file airflow.cfg in home directory.  
Folder for store tasks file is named dag in home directory.  
## Summary
This guide explains the deployment procedure conceptually. For detailed installation and configuration steps, 
see the Apache Airflow project documentation.
