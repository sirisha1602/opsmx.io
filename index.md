# Download and run AutoPilot CV

## Run OpsMx as a Container

Detailed below are the steps to turn on continuous verification for a dockerized deployment setup.
## Prerequisites
To remove any duplicate pod reference, execute the below
```yaml
	docker rm $(docker ps -a -f status=exited -q)
```

### 1. Download OpsMx from DockerHub

Download the latest OpsMx docker image from the dockerhub (hub.docker.com)
 The ID of the image is: `docker.io/opsmx11/autopilot:v0.9.16.201906051211`
 
 To pull the image, run 
 
```yaml
	docker pull docker.io/opsmx11/autopilot:v0.9.16.201906051211
```

### 2. Allocate a Volume for Persistent Storage

Allocate a volume on the VM to store data beyond the lifecycle of the container. 

<pre><code>docker volume create opsmxdata</code></pre>

### 3. Download Database image from DockerHub

Download the latest Database docker image from the dockerhub (hub.docker.com)
 The ID of the image is: `docker.io/opsmx11/autopilot:db-0.9`

 To pull the image, run
 
```yaml
	docker pull docker.io/opsmx11/autopilot:db-0.9
```

### 4. Run Database on a pre-specified local IP

Execute the below to ensure data lives as a separate module, the database is bound to a IP and our application connects to this container via this IP. 
```yaml
	docker run --name opsmxdb -itd -p 5432:5432 -v opsmxdata:/var/lib/postgresql docker.io/opsmx11/autopilot:db-0.9
```

### 5. Execute the below command to identify the Database IP Address
To ensure the right DB IP address is capture, makesure the DB Name in the above step #4 and in this steps remains the same, as opsmxdb.
```yaml
	databaseip=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' opsmxdb)
```

### 6. Run OpsMx on Docker linked to Database

Run the OpsMx(AutoPilot) image on the machine where Docker is installed
OpsMx requires ports 8090, 9090 and 8161 to be open. Ensure these are opened in the VM and in Docker

Also, point the volume created to the Database of OpsMx to ensure the durability of the data. 

```yaml
	docker run --name opsmx-autopilot  -itd -p 8090:8090 -p 9090:9090 -p 8161:8161 --add-host=opsmxdata:$databaseip  docker.io/opsmx11/autopilot:v0.9.16.201906051211
```

### 7. Verify the Container

Check whether this docker image is running as a container, via: `docker ps`
This should show up a running container with a container id of `opsmx11/autopilot:v0.9.16.201906051211`

Using the container Id, enter into the container
		```yaml
			sudo docker exec -it opsmx-autopilot /bin/bash restart_service.sh $databaseip
		```


### 8. Integration with HTTPS

### Generate self-signed keystore steps:
**Step 1:** Execute below command in the path: /home/ubuntu/ and answered for all the questions which it asks and remember the password.
```yaml
	keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650
```

**Step 2:** Update keystore details in the below places
Open /opt/opsmx/config.properties file and update the password by search the keyword with “server.ssl.key-store-password”.

Open the tomcat /conf/server.xml file and update the password by searching the keyword “keystorePass”.

### To access the UI for login and registration - 

 <pre><code> https://{ip}:8161/opsmx-analysis/public/login.html </code></pre>

This completes the setup and configuration of OpsMx container.

### Signup & Login

To access the UI for login and registration -

 <pre><code> https://{ip}:8161/opsmx-analysis/public/login.html </code></pre>


### 9. User Creation/Fetching Process
Here, we explore options for user creating/fetching methogs.
After downloading the containers, login to the AutoPilot container and navigate to the following location <b>‘/opt/opsmx/’</b> edit the file <b>‘config.properties’</b>.

In case, if LDAP is marked to ‘False’. We should create users manually.In case, if LDAP is set to ‘True’, user will be able to login with the LDAP credentials.
	
### 10. Integration with LDAP

!!! note
	If the Auth provider is not LDAP, then skip this step

If the Auth provider is LDAP then the LDAP server details to be configured in the global properties file.

Edit opsmx global properties file to enter the ldap details
		```yaml
			sudo vi /opt/opsmx/config.properties
		```
		
Edit the last few properties related to LDAP and point it to the right ldap instance.
#### ldap login attributes

isLdapAuthEnabled=true

ldap.url=ldap://<ldap_server_ip>:389 (example : ldap://35.227.87.101:389)

ldap.base.dn=<base_dn> (example : cn=Users,dc=local,dc=opsmx,dc=com)

ldap.user.filter.pattern=(&(objectclass=person)(cn=USERNAME))

ldap.admin.user.groups=Administrators,empdev

1. isLdapAuthEnabled: Enable the isLdapAuthEnabled flag to true. This will ensure the authentication is done against the given ldap server
2. ldap.url: Set this to the ldap server ip
3. ldap.base.dn: Set the base dn to the distinguished name of the realm under which the users present would be authenticated
4. ldap.user.filter.pattern: Set the objectclass which represents the user (example:person or user)
5. ldap.admin.user.groups: Define admin user groups for allowing to save or update the data source credentials



### 11. Access the Login Page

To access the UI for login and registration - 

 <pre><code> http://{ip}:8161/opsmx-analysis/public/login.html </code></pre>

This completes the setup and configuration of OpsMx container.

## Signup & Login

To access the UI for login and registration -

 <pre><code> http://{ip}:8161/opsmx-analysis/public/login.html </code></pre>

**1. Signup a new User**

!!! note
	If the user is authenticated skip this step


![Screenshot](img/Login-1.png)


**2. Login with this new User**

![Screenshot](img/Login-2.png)


## Configuring Cloud Credentials

!!! note
	This is only required if data is being pulled directly from the container/VM. Not mandatory.

If you are deploying your applications to  Kubernetes, AWS or GCP, enable read access to the OpsMx Analysis platform

To enable read access, perform the following tasks

**Step 1:**  Click on “SETUP” from the Main menu

**Step 2:**  Click on “CLOUD” tab

**Step 3:**  Click on “ADD” Button. Currently, OpsMx supports  “Kubernetes” and “AWS” or “GCP” providers.

**Step 4:**  For “Kubernetes” ,enter account name in textbox then upload the appropriate  kubernetes credentials file by clicking on Browse and Upload Buttons.

![Screenshot](img/cloud.png)

**Step 5:**   Configure the Cloud Read credentials

For enabling AWS cloud, specify the Access Key ID, Secret access key ID and the AWS Region (e.g., US-West).  Check out how to generate AWS access credentials section for help with creating keys with needed permissions.

For enabling Google Cloud Platform, upload the appropriate credentials file.

**Step 6:** Save the credentials.

For Saving the Cloud Credentials, when it Uploads it automatically saved.

To delete a saved credential, click on the Delete link under Action box.


## Configuring Monitoring Credentials

OpsMx Analysis Platform needs to be configured to access the monitoring metric store for the deployments to be able to analyze the services. Following are the monitoring metric store and log analysis methods supported

1.	Elastic Search
	
2.	AWS Cloud Watch
	
3.	GCP Stack Driver
	
4.	Newrelic
	
5.	Prometheus
	
6.	Datadog

## Logs and Metrics: What are they, and how do they help me?

Before trying to understand, in detailed about logs & metrics. Let’s try to understand individually about logs and Metrics.

## What are logs?

A log message is a system generated set of data when an event has happened to describe the event. In a log message is the log data. Log data are the details about the event such as a resource that was accessed, who accessed it, and the time. Each event in a system is going to have different sets of data in the message.

## What are Metrics?

While logs are about a specific event, metrics are a measurement at a point in time for the system. This unit of measure can have the value, timestamp, and identifier of what that value applies to (like a source or a tag). Logs may be collected any time an event takes place, but metrics are typically collected at fixed-time intervals. These are referred to as the resolution.

## Do we need both Logs & Metrics?

Yes, because each has its own importance. A log captures the information that is related to an event in the system. Whereas captures the measurement of the health of a system, if there is an impact caused due to the event.

## How does Elasticsearch work?

We can send data in the form of JSON documents to Elasticsearch using the API or ingestion tools such as Logstash and Amazon Kinesis Firehose. Elasticsearch automatically stores the original document and adds a searchable reference to the document in the cluster’s index. You can then search and retrieve the document using the Elasticsearch API. You can also use Kibana, an open-source visualization tool, with Elasticsearch to visualize your data and build interactive dashboards.

## Is Elasticsearch free?

Yes, Elasticsearch is a free, open source software. You can run Elasticsearch on-premises, on Amazon EC2, or on Amazon Elasticsearch Service. With on-premises or Amazon EC2 deployments, you are responsible for installing Elasticsearch and other necessary software, provisioning infrastructure, and managing the cluster. Amazon Elasticsearch Service, on the other hand, is a fully managed service, so you don’t have to worry about time-consuming cluster management tasks such as hardware provisioning, software patching, failure recovery, backups, and monitoring.


In this part of the document, lets explore the process to enable New Relic & Elastic Search

To enable read access to the monitoring metric store, perform the following tasks

**Step 1:**  Click on “SETUP” from the Main menu

**Step 2:**  Click on “DATA SOURCES” tab

**Step 3:**  Click on the “ADD”.  Currently, OpsMx support AWS CloudWatch, GCP StackDriver, Newrelic, Prometheus, Elasticsearch and Datadog monitoring tools.


Configuring Monitoring Credentials

**Step 4:**   Configure the Monitoring Credentials

For enabling Newrelic, specify the Account Name, Application Name, Application key.

For enabling Elastic Search, specify the following details Account Name, End Point, Username, Password, Scope Value and Response Keywords.

For enabling Prometheus, specify the Account Name, End Point, Username and Password.

For enabling AWS CloudWatch, specify the Access Key ID, Secret access key ID and the AWS Region (e.g., US-West).

For enabling Google Cloud Platform StackDriver, upload the appropriate credentials file.

For enabling Datadog, specify the API key and Application key.


**Step 5:** Save the monitoring credentials.

For Saving the Monitoring Credentials continue by clicking on “SAVE” Button.

To delete a saved credential, click on the Delete link under Action box.

![Screenshot](img/datasource1.png)

Add Auth-Token, Username and Passwords wherever applicable.

!!! note

	Scope refers to the key used to retrive a unique entity from the monitoring provider.
	An example for Elastic-Search is provided below. 

![Screenshot](img/data-source-elasticsearch.png) 

Provide the below details to enable elasticsearch on autopilot

1. Account Name (For e.g. opsmx-elk)
2. End Point (For e.g. http://192.158.0.1:9200)
3. User Name
4. Password
5. Scope (For e.g. container_id)
6. Response Keywords (For e.g. message,exception,stacktrace)