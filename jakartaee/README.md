# Deploying a Java/Jakarta EE Application to JBoss EAP on Azure App Service
This demo shows how you can deploy a Java/Jakarta EE application to Azure using JBoss EAP on App Service.

## Build the Project
Navigate to the project source `jakartaee` and build the application:

```
mvn clean package
```

## Run Locally (Optional)
Start the application and PostgreSQL database using Docker Compose:

```
# Best to run these to avoid resource conflicts
# docker system prune -a
# docker volume prune -a
# docker-compose down -v
docker-compose up --build
```

Once the application starts, it will be accessible at http://localhost:8080.

You can explore the REST API for the application:

```bash
curl -v -X POST http://localhost:8080/resources/todo -H "Content-Type: application/json" -d '
{
"description": "Test REST API",
"completed": "true"
}'

curl http://localhost:8080/resources/todo
```

## Setup JBoss EAP on App Service
* Go to the [Azure portal](http://portal.azure.com).
* Select 'Create a resource'. In the search box, enter and select 'Web App'. Hit create.
* Select todo-app-group-`<your suffix>` as the resource group and enter todo-jboss-app as application name. Choose Java 17 as your 
runtime stack and JBoss EAP 8 as the Java web server stack. You can optionally pick the free tier for your pricing plan.
* Click next until you reach the monitoring tab. If you want faster deployment, turn off Application Insights. You should definitely do 
this for the free tier where compute capacity is very limited.
* Finish creating the resource.

## Setup Environment Variables
* In the portal home, go to 'All resources'. Find and click on the App Service instance named todo-jboss-app. Open the 
Settings -> Environment variables panel.
* Add the following variables: POSTGRESQL_DB_URL=jdbc:postgresql://todo-db-`<your suffix>`.postgres.database.azure.com:5432/postgres, 
POSTGRESQL_DB_USER=postgres, POSTGRESQL_DB_PASSWORD=Secret123!.

## Start the Application on JBoss EAP on App Service
* Open a console and execute the following to log onto Azure.

	```
	az login
	```

* Open the [pom.xml](pom.xml) file and replace occurrences of `reza` with `<your suffix>`.
* You should note the pom.xml. In particular, we have included the configuration for the Azure Maven plugin we are going to use to deploy 
the application to JBoss EAP on App Service:

```xml
<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-webapp-maven-plugin</artifactId>
    <version>2.13.0</version>
    <configuration>
        <appName>todo-jboss-app</appName>
        <resourceGroup>todo-app-group-reza</resourceGroup>
        <javaVersion>Java 17</javaVersion>
        <webContainer>JBossEAP 8</webContainer>
        <appSettings>
            <!-- Increase the timeout -->
            <property>
                <name>WEBSITES_CONTAINER_START_TIME_LIMIT</name>
                <value>500</value>
            </property>
            <property>
	        <name>WEBSITE_SKIP_AUTOCONFIGURE_DATABASE</name>
	        <value>true</value>
            </property>
        </appSettings>
        <deployment>
            <resources>
                <resource>
                    <type>lib</type>
                    <directory>${project.build.directory}/dependencies</directory>
                    <includes>
                        <include>postgresql.jar</include>
                    </includes>
                </resource>
                <resource>
                    <type>script</type>
                    <directory>${project.basedir}/src/main/jboss</directory>
                    <includes>
                        <include>postgresql-module.xml</include>
                        <include>configure-data-source.cli</include>
                    </includes>
                </resource>
                <resource>
                    <type>startup</type>
                    <directory>${project.basedir}/src/main/jboss</directory>
                    <includes>
                        <include>startup.sh</include>
                    </includes>
                </resource>
                <resource>
                    <directory>${project.basedir}/target</directory>
                    <includes>
                        <include>todo.war</include>
                    </includes>
                </resource>
            </resources>
        </deployment>
    </configuration>
</plugin>
```

* Use Maven to deploy the application from the `jakartaee` directory:

```
mvn clean package azure-webapp:deploy
```

* Keep an eye on the console output. You will see the application deployment progress. It may take a while for the deployment to complete.
* Once successfully deployed, you can access the application through its public endpoint. To get the public endpoint, go to 
portal home -> 'All resources'. Find and click on the App Service instance named todo-jboss-app. Go to the overview panel and copy the 
default domain. The application will be available at a URL like: https://todo-jboss-app-suffix.azurewebsites.net.
* Once the application starts, you can test the REST service at the 
URL: https://todo-jboss-app-suffix.azurewebsites.net/resources/todo or via 
the React UI at https://todo-jboss-app-suffix.azurewebsites.net.
