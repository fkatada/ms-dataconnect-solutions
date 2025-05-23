# The Watercooler (JWC) App Service

## Running the Application in Azure
The Watercooler application is meant to be run as an Azure App Service, and is deployed as such by the deployment script.  
The AppService name (which determines the URL of the AppService UI) can be set during the initial deployment process.
The default value is `watercooler-app`, so the default URL is `https://watercooler-app.azurewebsites.net`.  
You can also find it by going to the deployment resource group, opening the jwc App Service and going to the Overview page.  
The application automatically gets started during deployment, however, you can also stop it, start it and restart it
from the App Service Overview page.

### User Authentication
The Watercooler AppService uses the [built-in authentication mechanism of the AppService](https://docs.microsoft.com/en-us/azure/app-service/overview-authentication-authorization).   
Anonymous access is not permitted.
In order to log in, you need to access the AppService UI URL and use your Azure ActiveDirectory identity to log in.


## Running the Application locally
Although a completely standalone local deployment is not possible (the app still needs to rely on certain services,
such as Active Directory, from an existing development deployment in Azure), it is useful to run the
app locally as this allows both fast development and testing of new features, as well as debugging in case of problems.

### Preconditions
The following steps need to be taken for the local app to be fully functional.
Some of these steps need to be performed after the local application was started, for it to be fully functional.

#### Defining a Run Configuration in your favorite IDE
We used IntelliJ IDEA for the development process, so that is what will be used as reference.
However, the equivalent steps can be done in your favorite IDE.
- Go to `Run` -> `Edit Configurations` -> Add New Configuration -> `Application` and choose a name for your configuration (e.g. `Watercooler`)
- Set the classpath of the `jwc-core` module
- Set `com.microsoft.graphdataconnect.watercooler.Runner` as the main class
- Set under `Environment variables` all the environment variables mentioned in the sections below


#### Mssql local database configuration
- In order to set up a local database,make sure that docker is running, and run the deployment/local/docker-compose.yml file
```shell script
cd deployment/local
docker-compose -f ./docker-compose.yml up -d
```
- Two docker containers will start each running a mssql server. One will be used for running the application locally, the other for running tests locally

- After the containers have started for the first time, run the following commands in order to create the `jwc_database` in each mssql server:
For linux/mac-os:
```shell script
docker exec -i $(docker ps -aqf "name=jwc_database$") /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'password_!23' -d master <./create-database.sql
docker exec -i $(docker ps -aqf "name=jwc_database-test$") /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'password_!23' -d master <./create-database.sql
```

For windows: execute `docker ps` and retrieve the container id-s for the `jwc_database` and `jwc_database-test`:
```commandline
docker exec -i <<container_id_for_jwc_database>> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "password_!23" -d master <./create-database.sql
docker exec -i <<container_id_for_jwc_database_test>> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "password_!23" -d master <./create-database.sql

```



Or you can use git-bash from here: https://git-scm.com/download/win and in the terminal provided by git-bash you can 
run the above mentioned docker commands with the linux syntax.

- The application will have to connect to a database that contains the data related to the watercooler groups and user profiles
  The following tables are necessary in order for the application to be fully functional:
  `configurations`, `employee_profile`, `groups_per_day`, `groups_per_week`, `members_group_personal_meetings`, `members_to_group_participation`.
  - In order to create tables run the following commands docker container  
      From project root directory, execute:    
      For Linux/mac-os:     
      ```
        docker exec -i $(docker ps -aqf "name=jwc_database$") /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'password_!23' -d jwc <./jwc/core/src/main/resources/db/migration/V0001__init.sql
        docker exec -i $(docker ps -aqf "name=jwc_database-test$") /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'password_!23' -d jwc <./jwc/core/src/main/resources/db/migration/V0001__init.sql
      ```
      For Windows:
      ```
        docker exec -i <<container_id_for_jwc_database>> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "password_!23" -d jwc <./jwc/core/src/main/resources/db/migration/V0001__init.sql
        docker exec -i <<container_id_for_jwc_database_test>> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "password_!23" -d jwc <./jwc/core/src/main/resources/db/migration/V0001__init.sql
      ```


## Environment variables  
The application uses a list of environment variables which allows the developer to adapt application's settings to his own development environment.  
Please note that some environment variables are mandatory to set. The application will start with default values but some functionality won't work.   
```
JWC_DATASOURCE_URL=jdbc:sqlserver://localhost:1433;databaseName=jwc;
```
 - The SQL server URL connection. Ip/domain, Port and database name can be changed.   

```
JWC_DATASOURCE_USERNAME=sa
```
 - The SQL server username. The server was initially configured to use 'sa' username

```
JWC_DATASOURCE_PASSWORD=password_!23
```
 - The SQL server password. The server was initially configured to accept this password


### Mandatory environment variables   
In order to set this values, you need to have already one active deployment as described in [Running the Application locally](#Running the Application locally) sections

```
AZURE_KEY_VAULT_URI=
```
 - The Azure KeyVault URL of your active deployment
 ![image](../../docs/imgs/copy_keyvault_url.png)
 
```
JWC_SERVICE_PRINCIPAL_SECRET=
```
- the secret of the service principal (app registration) used by the jwc App Service;
  the secret is stored in the App Key Vault, the name of the secret is jwc-jwc-service-principal-secret     
 ![image](../../docs/imgs/keyvault_service_principal_secret.png)
```
SERVICE_PRINCIPAL_TENANT_ID=
```
 - the tenant id of the service principal (app registration) used by the JWC App Service   
 ![image](../../docs/imgs/copy_tenant_id_from_app_service.png)

```
SERVICE_PRINCIPAL_CLIENT_ID=
```
 - the client id of the Web App service principal (app registration) used by the jwc App Service   
  ![image](../../docs/imgs/copy_service_pricipal_id.png)   
  
```
ADF_SUBSCRIPTION_ID=
```
 - the subscription id of the Azure Data Factory service   
   ![image](../../docs/imgs/copy_subscription_id.png)  

```
ADF_RESOURCE_GROUP_NAME=
```
 - the name of the resource group where the application is deployed   
 ![image](../../docs/imgs/copy_resource_group_name.png)  
 
```
ADF_NAME=
```
 - the name of the Azure Data Factory service name   
 ![image](../../docs/imgs/copy_adf_name.png)  
   
```
JWC_APPSERVICE_URL=   
```
- the url of the jwc App Service running in Azure  
 ![image](../../docs/imgs/copy_app_url.png)  



#### Copy the app service auth cookie locally
Since while running in an AppService the Watercooler app uses the built-in authentication mechanism of the AppService,
when running the application locally (i.e. outside the AppService), there is no authentication functionality to rely on.  
However, most of the functionality of the application is dependent on the identity of the current user, which is
derived by the application code from the `X-MS-CLIENT-PRINCIPAL-NAME` request header or, alternatively, from the
authentication cookie (`AppServiceAuthSession`). These get created by the AppService authentication process and are thus
not available locally.  
Therefore, while running locally you will need to explicitly provide your identity to the application, by manually  
setting the authentication cookie. This must be previously generated by logging into the development App Service

To retrieve the authentication cookie from the App Service:
1. open the Watercooler UI of the development App Service in the browser
    - Please see the section about [Running the Application in Azure](#running-the-application-in-azure) for details
      about how to obtain the URL of the AppService UI
2. log into the application if required
3. use the browser developer tools to obtain the cookie's value
    - in Chrome:
        - right click in the page -> `Inspect` -> go to the `Application` tab -> `Storage` -> `Cookies`
        - copy the whole value of the `AppServiceAuthSession` cookie
    - in Firefox
        - right click in the page -> `Inspect` -> go to the `Storage` tab -> `Cookies`
        - copy the whole value of the `AppServiceAuthSession` cookie

Set the cookie in the Watercooler application running on localhost:
1. after the local application is started, open the local Watercooler UI in the browser (localhost:7655)
2. use the browser developer tools to set the cookie's value
    - in Chrome:
        - right click in the page -> `Inspect` -> go to the `Application` tab -> `Storage` -> `Cookies`
        - create a cookie named `AppServiceAuthSession` and paste the copied string as its value
    - in Firefox
        - right click in the page -> `Inspect` -> go to the `Storage` tab -> `Cookies`
        - create a cookie named `AppServiceAuthSession` and paste the copied string as its value

> Note: The auth cookie from the App Service needs to be updated locally every time it got refreshed in the App Service
> (e.g. because the session expired).


### Starting the application from your IDE
Now that all the prerequisites have been met, start the previously defined run configuration from our IDE.  
E.g. in IntelliJ IDEA, start the `Watercooler` run configuration defined previously either in Run or Debug mode.

The following lines of code will have to be the first lines in each method receiving an `@RequestHeader httpHeaders: HttpHeaders` parameter:
```
httpHeaders.set("x-ms-client-principal", "<x-ms-client-principal-value>")
httpHeaders.set("x-ms-token-aad-id-token", "<x-ms-token-aad-id-token-value>")
httpHeaders.set("x-ms-token-aad-refresh-token", "<x-ms-token-aad-refresh-token-value>")
```

The value of the `x-ms-client-principal` , `x-ms-token-aad-id-token`, `x-ms-token-aad-refresh-token` headers can be determined from the jwc App Service logs
when `JWC_LOG_LEVEL` env variable value is `debug`. This has to be set from jwc App Service `Configurations` section.  
When `JWC_LOG_LEVEL` is configured as described above, every request will contain the authentication headers.

In order to use the "Ingestion Mode Switch" and "HR Data Upload" functionalities locally, the following env variables will have to be set as described in [Environment variables](#Environment variables) section
```
JWC_APPSERVICE_URL=<JWC_url>
```
```
SERVICE_PRINCIPAL_TENANT_ID=<tenant_id>  
```
```
SERVICE_PRINCIPAL_CLIENT_ID=<client_id> 
```
```
JWC_SERVICE_PRINCIPAL_SECRET=<sp_secret> 
```
```
ADF_SUBSCRIPTION_ID=<subscription_id> 
```
```
ADF_RESOURCE_GROUP_NAME=<resource_group_name>
```
```
ADF_NAME=<azure_data_factory_service_name>
```

### Running the tests locally
The jwc_database-test docker container needs to be running for the tests to work.  
The tests can then be run either using maven or the IDE.  
To run the tests via maven, simply run the following command from the `jwc` folder, or the specific module which you want to test
```
mvn test
```
Alternatively, the unit tests automatically get run when you build a module, as long as the `-DskipTests` flag is not used
```
mvn clean install
```


