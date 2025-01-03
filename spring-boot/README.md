# Deploying a Spring Boot Application to Java SE on Azure App Service
This demo shows how you can deploy a Spring Boot application to Azure using 
Java SE on App Service.

## Build the Project
Navigate to the project source `spring-boot` and build the application:

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

## Set up Java SE on App Service
* Go to the [Azure portal](http://portal.azure.com).
* Select 'Create a resource'. In the search box, enter and select 'Web App'. 
Hit create.
* Select todo-app-group-`<your suffix>` as the resource group and enter 
todo-spring-app as application name. Choose Java 17 as your 
runtime stack and Java SE as the Java web server stack. You can optionally pick the 
free tier for your pricing plan.
* Click next until you reach the monitoring tab. If you want faster deployment, 
turn off Application Insights. You should definitely do 
this for the free tier where compute capacity is very limited.
* Finish creating the resource.

## Set up Environment Variables
* In the portal home, go to 'All resources'. Find and click on the App Service instance named todo-spring-app. Open the Settings -> Environment variables panel.
* Add the following variables: SPRING_DATASOURCE_URL=jdbc:postgresql://todo-db-`<your suffix>`.postgres.database.azure.com:5432/postgres, SPRING_DATASOURCE_USERNAME=postgres, SPRING_DATASOURCE_PASSWORD=Secret123!.

## Start the Application on Java SE on App Service
* Open a console and execute the following to log onto Azure.

	```
	az login
	```

* Open the [pom.xml](pom.xml) file and replace occurrences of `reza` with `<your suffix>`.
* You should note the pom.xml. In particular, we have included the configuration for the Azure Maven plugin we are going to use to deploy 
the application to Java SE on App Service:

```xml
<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-webapp-maven-plugin</artifactId>
    <version>2.13.0</version>
    <configuration>
        <appName>todo-spring-app</appName>
        <resourceGroup>todo-app-group-reza</resourceGroup>
        <javaVersion>Java 17</javaVersion>
        <webContainer>Java SE</webContainer>
        <appSettings>
            <property>
	            <name>WEBSITE_SKIP_AUTOCONFIGURE_DATABASE</name>
	            <value>true</value>
            </property>
        </appSettings>
        <deployment>
            <resources>
                <resource>
                    <directory>${project.basedir}/target</directory>
                    <includes>
                        <include>todo.jar</include>
                    </includes>
                </resource>
            </resources>
        </deployment>
    </configuration>
</plugin>
```

* Use Maven to deploy the application from the `spring-boot` directory:

```
mvn clean package azure-webapp:deploy
```

* Keep an eye on the console output. You will see the application deployment progress. It may take a while for the deployment to complete.
* Once successfully deployed, you can access the application through its public endpoint. To get the public endpoint, go to 
portal home -> 'All resources'. Find and click on the App Service instance named todo-spring-app. Go to the overview panel and copy the 
default domain. The application will be available at a URL like: https://todo-spring-app-suffix.azurewebsites.net.
* Once the application starts, you can test the REST service at the 
URL: https://todo-spring-app-suffix.azurewebsites.net/resources/todo or via 
the React UI at https://todo-spring-app-suffix.azurewebsites.net.
