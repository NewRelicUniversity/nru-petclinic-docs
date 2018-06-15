Prerequisites
-------------
For ease of deployment, we run the sample application in `Docker <https://www.docker.com/>`_ containers. In order to deploy the application, you will need: 

* A host running MacOS or a Linux-based operating system with `Docker <https://www.docker.com/community-edition>`_ installed. We use a t2.small EC2 instance running at AWS. <The application may run under Docker on Windows, but we have not tested it. The deployment process for Windows may differ somewhat from these instructions>`_;

* A `New Relic Pro or Enterprise account <https://docs.newrelic.com/docs/accounts/install-new-relic/account-setup/create-your-new-relic-account>`_. 

Deployment Steps
----------------
1. **Create mapped directories.** Log into your host and create two directories: one for the New Relic agent and associated files, and the other for the Pet Clinic application itself. These directories will be mapped into the Docker container, allowing you to modify the application or New Relic configuration without rebuilding the container. 

 In this example, the directories are named `newrelic` and `webapps`, and are created in the home directory of the logged-in user:

 .. code-block:: bash

   $ mkdir newrelic
   $ mkdir webapps
 
2. **Download the New Relic Java agent.** While still logged into your host, execute the following commands to download and extract the New Relic Java agent. Replace the `{account-id}` placeholder with your New Relic account ID, and the X.X.X placeholder with the version number of the downloaded file:

 .. code-block:: bash

   $ wget https://rpm.newrelic.com/accounts/{account-id}/download_agent?lang=java
   $ unzip newrelic-java-X.X.X.zip
 
3. **Replace the default `newrelic.yml` configuration file.** Execute the following command to download a customized `newrelic.yml` file into the `newrelic` directory:

 .. code-block:: bash

   $ wget https://github.com/NewRelicUniversity/petclinic-deploy-docs/blob/master/newrelic.yml -O newrelic/newrelic.yml
 
 This file uses environment variables for the license key and app name, and disables the New Relic Java agent's `circuit breaker <https://docs.newrelic.com/docs/agents/java-agent/custom-instrumentation/circuit-breaker-java-custom-instrumentation>`_. 
 
4. **Download the Pet Clinic application.** Execute the following command to download the `petclinic.war` file into the `webapps` directory:

 .. code-block:: bash

   $ wget https://github.com/NewRelicUniversity/generate-apm-data/raw/master/petclinic-1.0.war -O webapps/petclinic.war
 
5. **Start the MySQL Docker container.** Execute the following command to create a Docker container running MySQL: 

 .. code-block:: bash

   $ docker run -d --name mysql \ 
       -e MYSQL_ROOT_PASSWORD="petclinic" \ 
       -e MYSQL_DATABASE="petclinic" \ 
       -e MYSQL_USER="petclinic" \ 
       -e MYSQL_PASSWORD="petclinic" \ 
       -p 3306:3306 mysql:5.5
 
6. **Start the Apache Tomcat Docker container.** Execute the following command to create a Docker container running Apache Tomcat. Replace the `{your-license-key}` placeholder with the license key of your New Relic account. If you wish, you may change the value of the `NEW_RELIC_APP_NAME` parameter: 

 .. code-block:: bash

   $ docker run -it -d --tmpfs /run --tmpfs /tmp --name petclinic \ 
       -e NEW_RELIC_APP_NAME="New Relic Pet Clinic" \ 
       -e JAVA_OPTS="-Xms128m -Xmx320m -XX:MaxPermSize=128m -javaagent:/usr/local/tomcat/newrelic/newrelic.jar" \ 
       -e JDBC_CONNECTION_STRING="jdbc:mysql://mysql:3306/petclinic" \ 
       -e NEW_RELIC_LICENSE_KEY="{your-license-key}" \ 
       -v ~/webapps:/usr/local/tomcat/webapps -v ~/newrelic:/usr/local/tomcat/newrelic \ 
       --link mysql:mysql -p 80:8080 tomcat:8.0

 The above command maps the `webapps` folder on your host machine to Tomcat's `webapps` folder inside the container; Tomcat should start the Pet Clinic application automatically.
 
After a few minutes, you should be able to access the Pet Clinic application at `http://{your-host-name}/petclinic`. 