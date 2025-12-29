author: Ayca Öner
id: mendix_rest_sql_connector
summary: Guide on how to use the Mendix REST SQL Connector to integrate Snowflake data into a Mendix application using REST-based SQL execution.
categories: connectors, partner-integrations
environments: web
status: Published
tags: Getting Started, Data Science, Data Engineering, Connectors, Native Apps, External Connectivity, Mendix

# An Introduction to the Mendix REST SQL Connector
<!-- ------------------------ -->
## Overview

Duration: 5 minutes

In this QuickStart, you will learn how to set up a secure connection to Snowflake, retrieve data, and integrate it into a Mendix application using the Snowflake REST SQL Connector.

This QuickStart assumes basic familiarity with Mendix Studio Pro, microflows, and SQL, as well as access to a Snowflake account.

[Mendix](https://www.mendix.com/snowflake/) is a low-code application development platform that enables the rapid creation of enterprise-grade applications. Compared to [Snowflake Native Apps](https://docs.snowflake.com/en/developer-guide/native-apps/native-apps-about) built with [Streamlit](https://docs.snowflake.com/en/developer-guide/streamlit/about-streamlit), Mendix focuses on building full-featured business applications with highly customizable user interfaces and support for multiple interaction channels, such as web and mobile applications.

Streamlit is primarily aimed at developers and data analysts and requires Python expertise to build dashboards and data applications. Mendix, by contrast, enables teams to build mobile-ready and responsive web applications without writing code, while still supporting complex workflows, role-based access, and multiple deployment options including public cloud, private cloud, on-premises, and edge deployments. In addition, the Mendix Marketplace provides a wide range of ready-to-use widgets and connectors.

![Mendix Snowflake Connectors](assets/mendix_snowflake_connectors.png)

Mendix provides two connectors for integrating with Snowflake:

- The [External Database Connector](https://marketplace.mendix.com/link/component/219862)
- The [Snowflake REST SQL Connector](https://marketplace.mendix.com/link/component/225717)

In addition, Mendix offers the Mendix Data Loader, which enables data ingestion from Mendix into Snowflake. For more information, see the [Mendix Data Loader QuickStart](https://QuickStarts.snowflake.com/guide/mendix_data_loader/index.html#0).

This QuickStart focuses specifically on the Snowflake REST SQL Connector.

The Snowflake REST SQL Connector allows you to authenticate using either key-pair authentication with an RSA key pair (PKCS #8 standard) or OAuth, and to execute SQL statements on Snowflake via REST calls from within a Mendix application. Using this connector, you can:

- Read data from Snowflake
- Write data to Snowflake
- Trigger Snowflake Cortex ML functions
- Use Snowflake Cortex LLM functions
- Use Snowflake Cortex Analyst

For Snowflake Cortex–related functionality, the Snowflake account must be located in a region where Snowflake Cortex and Arctic are available. For details, see the [Snowflake documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#label-cortex-llm-availability).

Cortex-specific examples are out of scope for this QuickStart and will be covered in a future guide. For more information, refer to the Mendix documentation [here](https://docs.mendix.com/appstore/connectors/snowflake/snowflake-rest-sql/#cortex-analyst).

### What You’ll Accomplish

In this QuickStart, you will:

- Establish a secure connection between a Mendix application and a Snowflake environment
- Execute single SQL statements in Snowflake from a Mendix application
- Read data from Snowflake and display it within a Mendix application
- Update data in Snowflake from a Mendix application

### Prerequisites

Before you begin, make sure you have the following:

- A Mendix account. You can sign up [here](https://signup.mendix.com/).
- Mendix Studio Pro version [9.24.2](https://marketplace.mendix.com/link/studiopro/9.24.2) or later.  
  For the purposes of this QuickStart, we recommend using Mendix Studio Pro version 9.24.2.
- A [Snowflake](https://www.snowflake.com/) account with permissions to execute SQL statements

### Support Resources

If you are new to Mendix, it is recommended to first complete the [Rapid Developer learning paths](https://academy.mendix.com/link/paths) available on the Mendix Academy.

Throughout this QuickStart, downloadable `.mpk` files are provided. These are Mendix application packages that contain all steps implemented up to the point where the download link is shown. The `.mpk` files can be opened using Mendix Studio Pro version 9.25.2 or later.

To import an `.mpk` file:

1. Download the `.mpk` file using the provided links in this QuickStart
2. Open Mendix Studio Pro
3. Click **Import App Package** and select a folder where the project should be stored

### What You’ll Build

By completing this QuickStart, you will build:

- A basic Mendix application that communicates with Snowflake using the Snowflake REST SQL Connector

<!-- ------------------------ -->

## Setting Up Your Snowflake Environment

Duration: 5 minutes

In the following steps, you will prepare a Snowflake environment that will be used throughout this QuickStart. You will create a database, schema, and table, and populate the table with sample data that can be read, updated, and displayed in a Mendix application.

This setup allows you to explore how Mendix and Snowflake work together by executing SQL statements through the Snowflake REST SQL Connector.

Run the following SQL statements in a Snowflake worksheet to create the required database objects and insert sample data:

```sql
CREATE OR REPLACE DATABASE DATABASE_QuickStart;

CREATE OR REPLACE SCHEMA DATABASE_QuickStart.SCHEMA_QuickStart;

CREATE OR REPLACE TABLE DATABASE_QuickStart.SCHEMA_QuickStart.EMPLOYEE_INFO (
  EMPLOYEE_ID INTEGER,
  NAME VARCHAR,
  SURNAME VARCHAR,
  DATE_OF_BIRTH TIMESTAMP_NTZ, 
  IS_ACTIVE_EMPLOYEE BOOLEAN
);

INSERT INTO DATABASE_QuickStart.SCHEMA_QuickStart.EMPLOYEE_INFO
  (EMPLOYEE_ID, NAME, SURNAME, DATE_OF_BIRTH, IS_ACTIVE_EMPLOYEE)
VALUES
  (1, 'John', 'Doe', to_timestamp_ntz('1985-06-15'), true),
  (2, 'Jane', 'Smith', to_timestamp_ntz('1990-02-25'), true),
  (3, 'Michael', 'Johnson', to_timestamp_ntz('1978-11-30'), false),
  (4, 'Emily', 'Davis', to_timestamp_ntz('1982-04-10'), true),
  (5, 'Robert', 'Miller', to_timestamp_ntz('1965-08-20'), false),
  (6, 'Jessica', 'Wilson', to_timestamp_ntz('1995-01-15'), true),
  (7, 'David', 'Moore', to_timestamp_ntz('1988-09-05'), true),
  (8, 'Sarah', 'Taylor', to_timestamp_ntz('1975-12-20'), false),
  (9, 'Chris', 'Anderson', to_timestamp_ntz('1992-07-30'), true),
  (10, 'Laura', 'Thomas', to_timestamp_ntz('1980-03-10'), false);
```

After executing these statements, your Snowflake environment will contain a sample `EMPLOYEE_INFO` table that will be used in subsequent steps of this QuickStart.

## Setting Up Your Mendix Environment

Duration: 10 minutes

Mendix provides a detailed QuickStart on [Building a Responsive Web App](https://docs.mendix.com/QuickStarts/responsive-web-app/). If you are new to Mendix, we recommend completing that QuickStart or keeping it open alongside this guide to look up any unfamiliar concepts referenced in the steps below.

1. Install Mendix Studio Pro:
   - **Windows users**: Download and install Mendix Studio Pro version 9.24.2 from [here](https://marketplace.mendix.com/link/studiopro/9.24.2).
   - **macOS users**: Download the latest version of Mendix Studio Pro from [here](https://marketplace.mendix.com/link/studiopro).

   You will need a [Mendix account](https://signup.mendix.com/) to use Mendix Studio Pro.

2. Open Mendix Studio Pro and create a new application. Select **Blank Web App** as the starting point.

3. Download the latest **Snowflake REST SQL Connector** from the Mendix Marketplace into your application.

![Mendix Marketplace](assets/mendix_marketplace.png)

After the download completes, some errors may appear because dependency modules of the Snowflake REST SQL Connector also need to be installed. These dependencies can be downloaded directly within Mendix Studio Pro and do not require any external installations.

4. Download the following additional modules from the Mendix Marketplace:

- [GenAI Commons](https://marketplace.mendix.com/link/component/227933)
- [Encryption](https://marketplace.mendix.com/link/component/1011)
- [Community Commons](https://marketplace.mendix.com/link/component/170)

To use the functionality provided by the Encryption module, you must configure the **EncryptionKey** and **EncryptionPrefix** constants. For more information, see the [Encryption module documentation](https://docs.mendix.com/appstore/modules/encryption/#configuration).

If you are not using Mendix Studio Pro version 9.24.2, you may encounter errors caused by App Store module migrations. Navigate to the **Errors** panel in Mendix Studio Pro and resolve the errors before continuing. In most cases, this can be done by right-clicking the error and selecting **Update widget**.

To use Snowflake capabilities in a Mendix application with the Snowflake REST SQL Connector, an authentication method must be configured. For demonstration purposes, this QuickStart uses **key-pair authentication**.

5. Configure key-pair authentication in Snowflake by following the Snowflake documentation:

- Generate a private key
- Generate a public key
- Assign the public key to a Snowflake user  

   See the [Snowflake key-pair authentication documentation](https://docs.snowflake.com/en/user-guide/key-pair-auth) for detailed instructions.
  
To simplify the configuration process in Mendix, the Snowflake REST SQL Connector provides ready-to-use pages and microflows that can be added to your application.

6. Set up key-pair authentication in Mendix:

- In the **App Explorer**, right-click the **MyFirstModule** module and select **Add page**.
- Create a new **Blank Page** and name it **Configuration_Page**.
- Open **Navigation** in the App Explorer and add the page to the navigation structure.
- Resolve the resulting error by opening the **Configuration_Page**, navigating to the page properties, and under **Navigation**, set **Visible for** to **User**, then click **OK**.
- In the **App Explorer**, under the **SnowflakeRESTSQL** module, locate the **SNIPPET_SnowflakeConfiguration** snippet and drag it onto the **Configuration_Page**.
- If application security is enabled, assign the module role **SnowflakeRESTSQL.Administrator** to the application roles that will configure the Snowflake connection.

Run the application and select **View App**.

Several `.mpk` files are provided with earlier steps already implemented. These can be used as reference material or to continue if you encounter issues. For security reasons, the private key required to connect to Snowflake is not included and must be configured manually in each `.mpk` file.

[Download first .mpk](assets/REST_SQL_QuickStart.mpk "download")

- ![Run Mendix Application](assets/run_application.png)
- Navigate to the page where you added the configuration snippet
- Click **New**
- On the **Connection details** page, complete all fields with your Snowflake account information:
  - `Name`: Identifier for the connection within the Mendix application. This value is not sent to Snowflake.
  - `AccountURL`: The Snowflake account URL used to connect to the Snowflake API (for example, `https://sdc-prd.snowflakecomputing.com`)
  - `ResourcePath`: The Snowflake API resource path, for example `/api/v2/statements`
  - `AccountIdentifier`: The unique identifier of your Snowflake account
  - `Username`: The Snowflake username used for authentication
- Enter the passphrase and upload your private key file in `.p8` format
- Click **Save** to store the connection, or **Save and test connection** to generate a JSON Web Token (JWT) and validate the connection

![Connection Details](assets/connection_details.png)

<!-- ------------------------ -->
## Getting to Know the Snowflake REST SQL Connector

Duration: 10 minutes

After configuring authentication for Snowflake, you can use the Snowflake REST SQL Connector to execute SQL statements from within your Mendix application by calling the provided microflow activities.

The primary microflow action used to execute SQL statements is **POST_v1_ExecuteStatement**, which is located in the **SnowflakeRESTSQL** module.

![Execute Statement](assets/execute_statement.png)

As shown in the image above, this microflow action requires three input parameters and returns a list of `HttpResponse` objects:

- **ConnectionDetails**  
  The object created in earlier steps that contains the Snowflake connection and authentication configuration.

- **Token**  
  The authentication token used to access Snowflake. This can be either a JWT or an OAuth token, depending on the configured authentication method.  
  In this QuickStart, key-pair authentication is used. The JWT token can be generated by calling the **ConnectionDetails_GenerateToken_JWT** microflow using the `ConnectionDetails` object.

- **Statement**  
  The object that contains all request-specific information for the SQL execution:
  - `SQLStatement`: The SQL statement to execute
  - `Timeout`: The number of seconds after which the request times out
  - `Database`: The Snowflake database to use
  - `Schema`: The Snowflake schema to use
  - `Warehouse`: The warehouse used for query execution
  - `Role`: The role with sufficient permissions to execute the SQL statement

When executed, this microflow sends a POST request to Snowflake containing the SQL statement and returns a list of `HttpResponse` objects that can be processed further within Mendix.

To demonstrate a complete implementation, the Snowflake REST SQL Connector includes an example microflow named **ExampleImplementation**. This microflow shows how to retrieve data from an existing Snowflake table and convert the response into Mendix objects.

![Example Implementation](assets/example_implementation.png)

The **ExampleImplementation** microflow uses the `TransformResponsesToMxObjects` activity to transform a list of `HttpResponse` objects into Mendix objects of a specified entity. This activity requires:

- A list of `HttpResponse` objects
- The target Mendix entity that should be created from the response data

It returns a list of Mendix objects of the selected entity type.

To support this example, an entity named **ExampleObject** is included in the domain model of the connector with the following attributes:

**ExampleObject**

- `ATTR_TXT` (String)
- `ATTR_INT` (Integer)
- `ATTR_LONG` (Long)
- `ATTR_BOOL` (Boolean)
- `ATTR_DECI` (Decimal)
- `ATTR_ENUM` (Enumeration)
- `ParsedDate` (Date and time)

This entity is provided purely as an example. When implementing your own use case, you must carefully define:

- Attribute names
- Data types
- Attribute order

These properties must match the structure of the data returned by Snowflake. The column order, column names, and data types in the SQL query result must align exactly with the selected Mendix entity.

For example, assume a Snowflake table contains the columns `column1` through `column8`. To retrieve data from this table and map it to `ExampleObject`, you must write an SQL query that aliases the columns to match the Mendix attribute names and data types, as shown below:

```sql
SELECT 
  column1 AS ATTR_TXT,
  column2 AS ATTR_INT,
  column3 AS ATTR_LONG,
  column4 AS ATTR_BOOL,
  column5 AS ATTR_DECI,
  column6 AS ATTR_ENUM
FROM your_table;
```

If the attribute names, data types, and column order match, the`TransformResponsesToMxObjects` activity automatically converts the query results into Mendix objects.

<!-- ------------------------ -->
## Presenting Snowflake Data in Mendix

Duration: 15 minutes

In Mendix, an [entity](https://docs.mendix.com/refguide/entities/) represents a class of real-world objects, while an instance of an entity is called an object. [Microflows](https://docs.mendix.com/refguide/microflows/) define the logic of your application. They allow you to perform actions and express logic visually, instead of writing traditional program code.

Using the information from the previous steps, you will now configure the application to retrieve data from the `EMPLOYEE_INFO` table created in the Snowflake environment and display it in a Mendix page.

### 1. Create the domain model

1. Open the domain model of your application and create an entity named **Table**.  
   Set the **Persistable** property to **No**.
2. Create another entity named **Employee**. This entity will represent the records retrieved from Snowflake.  
   Ensure that the attribute names, data types, and order match the columns of the `EMPLOYEE_INFO` table.  
   Set the **Persistable** property to **No**.

```sql
Employee
- Employee_id (Integer)
- Name (String)
- Surname (String)
- Date_of_Birth (Long)
- Is_Active_Employee (Boolean)
- Date_of_Birth_Parsed (Date and time)
```

3. Create a one-to-many association between **Table** and **Employee** by dragging the corner of the **Employee** entity to the corner of the **Table** entity.

### 2. Configure the microflow to retrieve data

![Table-Employee Association](assets/table_employee_association.png)
4. Configure the microflows required to retrieve data from Snowflake and prepare it for display:

- Duplicate the **EXAMPLE_ExecuteStatement** microflow into your module and rename it to **Employee_Retrieve**.
- Create a new microflow by right clicking the **MyFirstModule** module and choosing the **Add microflow** option. Name it **ACT_Employee_RetrieveAndShow**.
- Drag the **Employee_Retrieve** microflow into **ACT_Employee_RetrieveAndShow**.
- The first component in the **Employee_Retrieve** microflow is the **Create Statement** action. Let's edit this to be relevant to our needs.

Inside the **Employee_Retrieve** microflow:

- Edit the **Create Statement** action and configure it as follows:

```text
- SQLStatement: 'SELECT * FROM EMPLOYEE_INFO'
- Database: 'DATABASE_QuickStart'
- Schema: 'SCHEMA_QuickStart'
- Warehouse: <Desired warehouse>
- Role: <Snowflake role with sufficient privileges, for example 'ACCOUNTADMIN'>
```

- The second component is the **Retrieve ConnectionDetails** action. Configure this action to retrieve the authentication method created in Step 2.
  - XPath Constraint: [Name='*name_of_your_connection*']
- The third and fourth components retrieve the authentication token and execute the SQL statement in Snowflake. These components can remain unchanged.

- Before the **Transform Responses To MxObjects** action:
  - From the **Toolbox** pane on the right, add a **Create Object** action.
  - Select the **Table** entity and name the object **NewTable**.  
    All retrieved **Employee** objects will be associated with this **Table** object for display purposes.

- Configure the **Transform Responses To MxObjects** action:
  - Entity: **Employee**

- Inside the loop, configure the components to convert the `Date_of_Birth` value returned by Snowflake into a Mendix Date and time value and store it in `Date_of_Birth_Parsed`:
  - Update the **Create Variable** action with the following expression:

    ```
    addSeconds(dateTimeUTC(1970,1,1), $IteratorExampleObject/Date_of_Birth)
    ```

  - In the **Change Object** action:
    - Set the **ParsedDate** member to **Date_of_Birth_Parsed**
    - Click **New** to add a new member
    - Select the association **MyFirstModule.Employee_Table** and set its value to `$NewTable`

- Set the `$NewTable` variable as the return value of the microflow by right-clicking the **Create Table** action and selecting **Set $NewTable as return value**.
  
![Retrieve Employee Info](assets/Employee_Retrieve.png)
5. As the microflow is now almost complete, prepare the user interface to display the data.

- Create a new **Blank Page** named **Table_Display**.
- Drag a **Data view** from the **Toolbox** onto the page and select the **Table** entity as the data source from context.
- From the **Toolbox**, drag a **Data grid** into the **Table** data view.

![Add Data Grid](assets/add_datagrid.png)

- Open the Data grid properties and go to the **Data source** tab.
  - Change the type to **Association**
  - Select the **Employee** entity through the association
  - Click **OK**

![Select Employee](assets/datagrid_select_employee.png)

- When prompted, choose **Yes** to automatically generate the contents of the data grid.

6. Now that both the microflow and the page are configured, add a button to trigger the retrieval and open the display page.

- Open the **Home_Web** page.
- Add a **Call microflow button** from the **Toolbox**.
- Select **ACT_Employee_RetrieveAndShow** as the microflow.
- Set the button caption to **Retrieve and Show Employee Info**.

7. Open the **ACT_Employee_RetrieveAndShow** microflow:
- Select the **Employee_Retrieve** microflow call.
- In the **Output** section, set the object name to **Table** and click **OK**.
- Add a **Show page** action at the end of the microflow.
- Select the **Table_Display** page and click **OK**.

![Retrieve and show Employee](assets/ACT_Employee_RetrieveAndShow.png)

8. Resolve the remaining security errors:
- Grant **User** role access to the **ACT_Employee_RetrieveAndShow** microflow.
- Open the **Employee** entity and create access rules granting **Read** access to all attributes.
- For the **Table** entity, grant **Create** access to resolve the remaining errors.

9. Run the application and click **Retrieve and Show Employee Info** to retrieve and display employee information from Snowflake.

![Employee Table](assets/table_display.png)

[Download second .mpk](assets/REST_SQL_QuickStart_2.mpk "download")

<!-- ------------------------ -->
## Updating Snowflake Data from Within Mendix

Duration: 15 minutes
Now, we will extend our module to be able to edit the existing data in Snowflake.

1. Go to the display page. Above the attribute columns in the a white area, right-click and choose *Add button->Action*
![Add Button](assets/datagrid_add_button.png)
2. Double-click on this new button and change the caption to "Edit". Change the **On-click** event to *Show a page*. Click on **New**, change the name of the new page to be "Employee_Edit", go to section **Forms** and select **Form Vertical**.
![Edit Button](assets/datagrid_edit_button.png)
3. Open the new page and delete the text box for "Employee_ID". This is a unique ID that will be given to employees and should never be edited.
4. Next, delete the text box for "Date_Of_Birth". This is a long value that is retrived from Snowflake which is then converted to Date and time. We will use the "Date_Of_Birth_Parsed" and convert it to the correct format in the microflow that will be triggered.
5. Navigate to the *Properties* of the page and in the *Navigation* section for *Visible for* select *User* to give the user access to the page and solve the security error.
6. We need to now configure the microflow that will enable us to update the information in the Snowflake table.

- Duplicate the **EXAMPLE_ExecuteStatement** microflow into your module and rename it to something like *Employee_Update*
- Add a **Parameter** above the microflow from the **Toolbox**. **Name** is *Employee* and as the **Data type** keep *Object*. Click on **Select** and choose *Employee* as the entity.
- The first component in the microflow is the *Create Statement* action. Let's edit this to be relevant to our needs.

```sql
  - SQLStatement: 
  'UPDATE EMPLOYEE_INFO SET 
    NAME = ''' + $Employee/Name+ ''', 
    SURNAME = ''' + $Employee/Surname+ ''', 
    DATE_OF_BIRTH = to_timestamp_ntz(''' + formatDateTime($Employee/Date_of_Birth_Parsed, 'yyyy-MM-dd') + '''), 
    IS_ACTIVE_EMPLOYEE = ''' + toString($Employee/Is_Active_Employee) + '''  
   WHERE EMPLOYEE_ID = ' + $Employee/Employee_ID + ';'
  - Database: 'DATABASE_QuickStart'
  - Schema: 'SCHEMA_QuickStart'
  - Warehouse: *Desired warehouse*
  - Role: *Snowflake role with sufficient rights to execute statement*
```

- The second component is the *Retrieve ConnectionDetails* action. We will also need to configure this to retrieve the authentication method we created on Step 2
  - XPath Constraint: [Name='*name_of_your_connection*']
- The third and fourth components are to retrieve the authentication token and execute the statement in Snowflake and can stay as they are.
- The rest of the components can be deleted.
    ![ACT_Employee_Update](assets/Employee_Update.png)

7. Open the "Employee_Edit" page and double-click on the **Save** button. Change the **On-click** event to *Call a microflow* and create a new microflow called *ACT_Employee_Update* so that this microflow will be triggered whenever the information is changed and the **Save** button is clicked.
8. Drag *Employee_Update* into *ACT_Employee_Update* from the app explorer and after that drag the *Employee_Retrieve* microflow in there as well. Call the return value of the *Employee_Retrieve* microflow call to *Table*. Add a **Show page** action at the end of the *ACT_Employee_Update* microflow and make it call the *Table_Display* page.
   ![ACT_Employee_Update](assets/ACT_Employee_Update.png)
9. Run the application and test the functionalities of these buttons to update information in your Snowflake environment.

![Edit Employee Info](assets/table_display.png)

[Download third .mpk](assets/REST_SQL_QuickStart_3.mpk "download")

<!-- ------------------------ -->
## (Optional) Deploy the Mendix Application

Duration: 10

[Mendix Cloud](https://docs.mendix.com/developerportal/deploy/mendix-cloud-deploy/) is the default [deployment](https://docs.mendix.com/developerportal/deploy/mendix-cloud-deploy/deploying-an-app/) option for Mendix applications. It is a public cloud service for Mendix applications, with infrastructure built and maintained by Mendix and built on top of Amazon Web Services (AWS). The application you created can be be deployed on a free cloud sandbox environment to gain access to it from different browsers on the web and mobile. However, as the application and data on the application will be shared to the public, we always recommend security first be set up on the application before a deployment is done. You can implement security on the application yourself by using the [*Security*](https://docs.mendix.com/refguide/security/) Mendix documentation or you can install a security set up version here. (TODO: ADD FILE TO DOWNLOAD!!)

1. (Optional) Download the mpk file that includes the previous implemented steps as well as security. (TODO ADD FILE)
2. (Optional) Once downloaded, execute the file titled `SnowflakeRESTSQL-QuickStart.mpk`, a window prompt should appear
3. (Optional) Create a new folder and select it to unpack the project files. After unpacking, the project should appear in Mendix Studio Pro version 9.24
4. Inside Mendix Studio Pro, navigate to `Version Control`, then click `Upload to Version Control Server...` and confirm by clicking `OK`. A window titled Upload App to Team Server should appear
5. After the project has been uploaded to version control server, click `Publish`
6. After a while a Snackbar notification is displayed `Your application is published`
7. Click `View App` to see the login screen for your Mendix application

- (Optional) To log into your Mendix application
  - Use the username `demo_administrator`
  - To retrieve the password for this user inside Mendix Studio Pro, navigate to `App 'SFShowcase'` -> `Security` -> `Demo users` -> `demo_administrator` and then click the link that reads `Copy password to clipboard`

8. Save the endpoint of your Mendix application, you'll need it later

- Save `https://snowflakerestsql-QuickStart-sandbox.mxapps.io/` if your endpoint is `https://snowflakerestsql-QuickStart-sandbox.mxapps.io/login.html?profile=Responsive` 
- You have successfully deployed the Snowflake Showcase App onto a free cloud sandbox environment!

![Deploying your Mendix application](assets/publish.png)
9. Add your authentication method in the homepage after logging in.
10. Go back to Mendix Studio Pro and configure your Snowflake information in *Employee_Retrieve* and *Employee_Update*. 
  
![Configure Microflows](assets/configure_snowflake_info.png)
11. Click on publish again to deploy the application with the lates changes.

You can now use the microflows to retrieve and update Snowflake data from within Mendix.

<!-- ------------------------ -->
## Conclusion and Resources

Congratulations! You've successfully used the Snowflake REST SQL Connector and executed SQL statements in Snowflake from within a Mendix application.

If you’d like to learn more about Mendix please check out our [Rapid Developer Course](https://academy.mendix.com/link/paths/31/Become-a-Rapid-Developer) or explore other [learning paths](https://academy.mendix.com/link/home).

### What You Learned

- How to quickly configure the Snowflake REST SQL Connector in a Mendix application.
- How to execute SQL statements in Snowflake from within a Mendix application.

### Related Resources

- [Snowflake REST SQL Connector documentation](https://docs.mendix.com/appstore/connectors/snowflake/snowflake-rest-sql/)
- [What is Mendix](https://www.mendix.com/)
- [Snowflake REST SQL Connector Listing](https://marketplace.mendix.com/link/component/225717)
