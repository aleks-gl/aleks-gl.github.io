# Apache Airflow Installation Concept
## Introduction
This manual contains common description of Apache Airflow installation process for Linux based operating system. 
Presupposes that those interested in this document are familiar with the Apache Airflow tool, however, we will give 
a brief reference about the purpose and capabilities of this software.  

### Brief reference  
*Apache Airflow* — it is platform for programmatically manage of workflows, in other words – for run, schedule 
and control sequences of processes. Such us Apache Airflow has GUI it can be good solution for administrators, 
and others, that needed for tool for run and control of process sequence. Workflow in Airflow is presented as as 
Directed Acyclic Graphs. This means that we have well formed and controlled sequence of tasks.  
In other side, Airflow can be good solution such as it is can use different type of operations for task in workflow. 
For example, bash script or python code can be used as task. This product is open source project of Apache Software Foundation. 
More information about this tool can founded in documentation for Airflow.  
Purpose of this document is define recommendation for deployment Apache Airflow on Linux based systems 
(prefer for Ubuntu and RedHat Linux).

## Preparation
Number of using conditions must be refined to create precise description of deployment task. Before starting deploy 
we should have explicit definitions for:
1. application area of the product and target environment for product application must be defined
2. system architecture
3. security politics and rules.
   All of this points must be defined as architect concept of system and now we expose only relations between above 
   listed points and configuration steps.  
###   Application area and target environment
   Listed below configurations is depending of the application area.
1. **Apache Airflow is used in testing area or for development**.  
   When tool used in test area the system have no high load but frequent configuration changes occur and 
   the system requires the ability to restore the original state. 
   In addition, the system must be able to install additional modules (extra). 
   In this case, the set of stored technological data is not important and it is possible to use SQLite database 
   and default settings  ofthe Apache Airflow. Basic configuration parameters, such as the path to the repository
   with tasks, are changed by scripts for deploying the application.
2. **Apache Airflow is used in productive space**.  
   In this case, the system is subjected to consistently high loads and rare changes. 
   In addition, it is assumed that the system must have a fault-tolerant configuration. 
   If the system is used in the productive area, we must use a special assembly for the Docker image and PostgreSQL DBMS.
###   System architecture
   The target infrastructure configuration accordingly determines how the Apache Airflow platform is deployed. 
   For the purposes of this tutorial, we assume the deployment of the Apache Airflow software in the standard versions 
   offered in the documentation for this product:
1) Installing Apache Airflow as standalone instance or named local installation. 
   This option can be used both for installation on a physical dedicated server and for a Linux server running on a virtual machine.
2) Installing Apache Airflow as Docker image.  

###   Security politics and rules
   Because security configuration should be carried out on the basis of precise recommendations built on  
   company's security policy, so in this manual, we will clarify only the main aspects that should be paid attention to. 
   Apache Airflow documentation contain precise description of user roles, that used by default. 
   In this manual we suppose that default roles is used for create conception of security configuration.
   Function for create specific roles for adjust security also available in Apache Airflow.
   At this point we should define roles and access rights for them. As example we can define next roles:  
1. ***Admin*** – users, that have access to edit configurations and manage tasks
2. ***Users*** – some peoples that have acess rights for start or stop tasks and view process status
3. ***Viewer*** – those role means only visual control of process status.
   Also we can limit all access rights for others (***Public***).  
   
   *Technical tips:*  
   Entry point for define security configuration is file *airflow.cfg* in home directory.
   Specification of access rules is defined in *airflow/www/security.py*.  
###   Package specification
   From documentation for Apache Airflow we get next recommendation for software:  
       - Python: 3.6, 3.7, 3.8  
   Databases:  
   * PostgreSQL: 9.6, 10, 11, 12, 13
   * MySQL: 5.7, 8
   * SQLite: 3.15.0+  
   Kubernetes: 1.16.9, 1.17.5, 1.18.6  
   For the purposes of this tutorial we approve next configurations:  
   DBMS for Airflow:  
   * PostgreSQL 11.* must used in productive area;
   * SQLite can used in area for development and tests.  
   kubernetes — we can use any of the versions recommended for Apache Airflow and available within the target infrastructure.  
   MySQL DBMS version is not implied, both due to limitations for creating parallel executors and limitations 
   when creating a custom image for Docker.
###   Apache Airflow deployment types
   In the case of a typed set of packages that should go into the deployed system, 
   we have several deployment options available, namely:
   * local installation of Apache Airflow for a test environment (or development environment) with SQLite DBMS;
   * installing Apache Airflow as a Docker image for a test environment (or development environment) with SQLite DBMS;
   * local installation Apache Airflow for a test environment (or development environment) with PostgreSQL DBMS;
   * installing Apache Airflow as a Docker image for a test environment (or development environment) with PostgreSQL DBMS;
   * local installation Apache Airflow for a productive environment with PostgreSQL DBMS;
   * installing Apache Airflow as a Docker image for a productive environment with PostgreSQL DBMS.  

For each of listed variants we have also additional division to:
   * using Apache Airflow as dedicated server for workflow management;
   * using Apache Airflow in cluster.  
Adjustment for both of configuration types for use as part of a distributed system or as a dedicated server 
     should stored in configuration files and it should be fixed in installation scripts.  
     
*Practical tips:*  
Entry point for define configuration it is file *airflow.cfg* in home directory.
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
[  
sudo apt-get install -y --no-install-recommends \
freetds-bin krb5-user ldap-utils libffi6 libsasl2-2 libsasl2-modules libssl1.1 locales  lsb-release sasl2-bin unixodbc python3.6-dev openssl sqlite sqlite-dev default-libmysqlclient-dev libmysqld-dev postgresql  
]  

Accordingly, other Linux distributions use their own package managers, for example, for RedHat, Fedora and CentOS, 
in this case, you need to use the yum package manager, for SUSE, zypper is used.

### Apache Airflow installation.
#### Plain installation
If there is no need for a specialized version (having fixes in the source code or a version of the application 
that differs from the one supplied by default), a quick installation is possible using pip for the dedicated server 
option or docker to get an image.  
For *local install* of Apache Airflow will run next Bash script:  

*AIRFLOW_VERSION=2.0.1
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
EXTRAS=”postgres,google”
pip install "apache-airflow[${EXTRAS}]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"*    

So, variable EXTRAS will contain list of additional packages, that listed below.  

***Additional packages:***  
airbyte, all, all_dbs, amazon, apache.atlas, apache.beam, apache.cassandra, apache.druid,apache.hdfs,
apache.hive, apache.kylin, apache.livy, apache.pig, apache.pinot, apache.spark,apache.sqoop, apache.webhdfs, 
async, atlas, aws, azure, cassandra, celery, cgroups, cloudant,cncf.kubernetes, crypto, dask, databricks, 
datadog, devel, devel_all, devel_ci, devel_hadoop,dingding, discord, doc, docker, druid, elasticsearch, exasol, 
facebook, ftp, gcp, gcp_api,github_enterprise, google, google_auth, grpc, hashicorp, hdfs, hive, http, imap, jdbc, 
jenkins,jira, kerberos, kubernetes, ldap, microsoft.azure, microsoft.mssql, microsoft.winrm, mongo, mssql,mysql, 
neo4j, odbc, openfaas, opsgenie, oracle, pagerduty, papermill, password, pinot, plexus,postgres, presto, qds, qubole, 
rabbitmq, redis, s3, salesforce, samba, segment, sendgrid, sentry,sftp, singularity, slack, snowflake, spark, sqlite,
ssh, statsd, tableau, telegram, vertica,virtualenv, webhdfs, winrm, yandex, zendesk.  

To install all available packages, you can set the parameter EXTRAS=”all”. Accordingly, for a production environment, 
it is not recommended to install all packages and it is necessary to list the list of packages that will be used.  

***Plain install of Docket image***  
To get an image from a public register, run the command:

*docker pull apache/airflow*

#### Special version installation
The following steps describe how to get the Apache Airflow source code from the 
https://github.com/apache/airflow.git repository and build and install it. 
The following method involves more steps, but it has a number of advantages. When installing from the source code, 
you can select a version (for this you need to get the necessary git branch) 
and / or modify the source code if necessary.  
Download the Apache Airflow source code from the repository:
git clone https://github.com/apache/airflow.git  
If there is no need to install additional packages - just use pip to install the downloaded package:  
*pip install .*
or
*pip setup.py install*
If you need to install additional packages, list them in square brackets:  
*pip install .[async,google,amazon] --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-master/constraints-3.6.txt"*

***Build specialized version of Docker image***  
Building an image for CI can be done using the breeze command that included in Apache Airflow source package. 
From root of the source code directory run the command:  
*./breeze build-image*  
In order to build an image for use in a production environment, you need to clarify the version of the assembly:  
*./breeze build-image —production-image*  
Accordingly, when building an image, as well as during a local installation, you must specify a list of additional 
packages:  
*./breeze build-image --python 3.7 --extras "all" --production-image*  

## Initialization of Apache Airflow
### Database initialization.
Before starting the Apache Airflow software, you need to initialize the data by the steps briefly described below. Before performing the initial data initialization, you need to have connection to the data source, directories must be created and user rights assigned to these directories.
In the case of a local installation, we execute the script to initialize data and launch application:

*export AIRFLOW_HOME=~/airflow
AIRFLOW_VERSION=2.0.1
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"*  

Initialize the database
airflow db init  

Create users  

*airflow users create \
--username admin \
--firstname Peter \
--lastname Parker \
--role Admin \
--email spiderman@superhero.org*

 Start the web server, default port is 8080
*airflow webserver --port 8080*

Start the scheduler  
Open a new terminal or else run webserver with ``-D`` option to run it as a daemon  
*airflow scheduler*

So, at this point we have runned Apache Airflow and we can login into server at localhost:8080.  

In the case of start Apache Airflow as Docker image, first create the necessary directories in the home folder:  
*mkdir ./dags ./logs ./plugins*  
*echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env*  
On the system with the Apache Airflow image installed, run the command to start the data initialization procedure.  
*docker-compose up airflow-init*  
After completing the data initialization procedure, you can start the services.  
*docker-compose up*
## Final configuration of application
The final configuration of Apache Airflow implies performing actions that, for example, should provide:  
* logging and monitoring of application
* create of connections for interaction with served or processed systems
* writing code to define workflows
* procedures for integration into infrastructure, such as API integration for control and messaging.  

*Practical tips:*   
Entry point for define configuration it is file airflow.cfg in home directory.  
Folder for store tasks file is named dag in home directory.  
## Summary
This guide explains the deployment procedure conceptually. 
For detailed installation and configuration steps, see the Apache Airflow project documentation.