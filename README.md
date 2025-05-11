The client is a Real Estate Interactive Technology firm based in Canada. They wanted to build an interactive application to dynamically customize housing floor plans to delight their customers. They had a flash based tool which they needed to re-platform into HTML5 based interactive platform. For this, they wanted a re-engineering solution for their legacy product suite. To maintain huge critical historical data from legacy systems, client wanted a cloud-based solution.

The key challenge meant in re-engineering legacy systems as they were built in VB6 and outdated technology. As HTML provide limited events like single click, double click and right click, client wanted to re-platform it to more usable and interactive design solution. The client required a communicable design framework with more availability of design functions.

Thanks for laying out the scenario so clearly—this is a classic legacy modernization use case, with an interesting twist around UI interactivity and historical data preservation. Below is a complete solution design that covers the **SQL database tables**, **Azure components** (Service Bus, Storage), and a narrative around the **business and technical scenario**.

---

## ✅ **Scenario Summary**

A **Canadian Real Estate Interactive Technology firm** wants to:

1. **Re-platform a Flash-based housing floor plan customization tool** into an **HTML5 interactive application**.
2. **Re-engineer VB6-based legacy systems**.
3. **Preserve historical critical data**, by moving it to the **cloud**.
4. Enhance the interactivity of HTML5 which is otherwise limited in user event flexibility.

---

## 🚀 **Proposed Architecture (Modern Azure-Based Re-Engineering Solution)**

```
Web Frontend (HTML5 + JS Framework)
     |
     | REST API / SignalR (for real-time updates)
     |
Azure App Service (Web App / Azure Function App)
     |
     +--> Azure SQL Database (Normalized Schema)
     |
     +--> Azure Service Bus (Queue + Topics for async events)
     |
     +--> Azure Storage (Blob, Table for floor plan files and meta)
     |
     +--> Azure Data Factory (for legacy data import/migration)
     |
     +--> Azure Monitor / Application Insights (Telemetry)
```
Here’s the **Azure architecture diagram** visualizing the flow from the HTML5 UI to all Azure services involved in the re-platformed interactive housing floor plan solution.


````mermaid
graph TD
    A[User HTML5 Web UI] --> B[Azure App Service / Function App]
    B --> C[Azure SQL Database]
    B --> D[Azure Service Bus Queue/Topic]
    B --> E[Azure Blob Storage Floor Plan Files]
    D --> F[Azure Table Storage Event Logs]
    D --> E
    D --> C
    G[Azure Data Factory Legacy Import] --> C
    H[Azure Monitor / App Insights] --> B
````


---
Below is a set of SQL Server **stored procedures** using `UNIQUEIDENTIFIER` as the primary key type for the four tables:

---

## 🛠️ SQL Setup (Tables with `UNIQUEIDENTIFIER`)

Before the stored procedures, here’s a refined version of the table schema using `UNIQUEIDENTIFIER`:

```sql
-- Users table (referenced by Projects)
CREATE TABLE Users (
    UserID UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Username NVARCHAR(50) NOT NULL,
    Email NVARCHAR(100) UNIQUE NOT NULL,
    Role NVARCHAR(20),
    CreatedAt DATETIME DEFAULT GETDATE()
);

CREATE TABLE Projects (
    ProjectID UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    UserID UNIQUEIDENTIFIER FOREIGN KEY REFERENCES Users(UserID),
    ProjectName NVARCHAR(100),
    Description NVARCHAR(500),
    CreatedAt DATETIME DEFAULT GETDATE(),
    LastModified DATETIME
);

CREATE TABLE FloorPlans (
    FloorPlanID UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    ProjectID UNIQUEIDENTIFIER FOREIGN KEY REFERENCES Projects(ProjectID),
    FloorPlanName NVARCHAR(100),
    BaseFilePath NVARCHAR(200),
    ThumbnailUrl NVARCHAR(200),
    CreatedAt DATETIME DEFAULT GETDATE()
);

CREATE TABLE Customizations (
    CustomizationID UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    FloorPlanID UNIQUEIDENTIFIER FOREIGN KEY REFERENCES FloorPlans(FloorPlanID),
    ComponentType NVARCHAR(50),
    Properties NVARCHAR(MAX),
    PositionX FLOAT,
    PositionY FLOAT,
    CreatedAt DATETIME DEFAULT GETDATE()
);

CREATE TABLE LegacyDataAudit (
    AuditID UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    LegacySystemID UNIQUEIDENTIFIER,
    DataType NVARCHAR(50),
    SourceFileName NVARCHAR(100),
    ImportedBy NVARCHAR(100),
    ImportDate DATETIME DEFAULT GETDATE(),
    Status NVARCHAR(20)
);
```

---

## 🔄 Stored Procedures

### ✅ `Projects` CRUD

```sql
-- Create
CREATE PROCEDURE usp_Project_Create
    @UserID UNIQUEIDENTIFIER,
    @ProjectName NVARCHAR(100),
    @Description NVARCHAR(500),
    @ProjectID UNIQUEIDENTIFIER = NULL
AS
BEGIN
     SET @ProjectID = NEWID();
     INSERT INTO Projects (ProjectID, UserID, ProjectName, Description, LastModified)
     VALUES (@ProjectID, @UserID, @ProjectName, @Description, GETDATE());
     SELECT * FROM LegacyDataAudit WHERE ProjectID = @ProjectID;
END;

-- Read
CREATE PROCEDURE usp_Project_Read
    @ProjectID UNIQUEIDENTIFIER
AS
BEGIN
    SELECT * FROM Projects WHERE ProjectID = @ProjectID;
END;

-- Update
CREATE PROCEDURE usp_Project_Update
    @ProjectID UNIQUEIDENTIFIER,
    @ProjectName NVARCHAR(100),
    @Description NVARCHAR(500)
AS
BEGIN
    UPDATE Projects
    SET ProjectName = @ProjectName,
        Description = @Description,
        LastModified = GETDATE()
    WHERE ProjectID = @ProjectID;
END;

-- Delete
CREATE PROCEDURE usp_Project_Delete
    @ProjectID UNIQUEIDENTIFIER
AS
BEGIN
    DELETE FROM Projects WHERE ProjectID = @ProjectID;
END;
```

---

### ✅ `FloorPlans` CRUD

```sql
-- Create
CREATE PROCEDURE usp_FloorPlan_Create
    @ProjectID UNIQUEIDENTIFIER,
    @FloorPlanName NVARCHAR(100),
    @BaseFilePath NVARCHAR(200),
    @ThumbnailUrl NVARCHAR(200)
AS
BEGIN
    INSERT INTO FloorPlans (FloorPlanID, ProjectID, FloorPlanName, BaseFilePath, ThumbnailUrl)
    VALUES (NEWID(), @ProjectID, @FloorPlanName, @BaseFilePath, @ThumbnailUrl);
END;

-- Read
CREATE PROCEDURE usp_FloorPlan_Read
    @FloorPlanID UNIQUEIDENTIFIER
AS
BEGIN
    SELECT * FROM FloorPlans WHERE FloorPlanID = @FloorPlanID;
END;

-- Update
CREATE PROCEDURE usp_FloorPlan_Update
    @FloorPlanID UNIQUEIDENTIFIER,
    @FloorPlanName NVARCHAR(100),
    @BaseFilePath NVARCHAR(200),
    @ThumbnailUrl NVARCHAR(200)
AS
BEGIN
    UPDATE FloorPlans
    SET FloorPlanName = @FloorPlanName,
        BaseFilePath = @BaseFilePath,
        ThumbnailUrl = @ThumbnailUrl
    WHERE FloorPlanID = @FloorPlanID;
END;

-- Delete
CREATE PROCEDURE usp_FloorPlan_Delete
    @FloorPlanID UNIQUEIDENTIFIER
AS
BEGIN
    DELETE FROM FloorPlans WHERE FloorPlanID = @FloorPlanID;
END;
```

---

### ✅ `Customizations` CRUD

```sql
-- Create
CREATE PROCEDURE usp_Customization_Create
    @FloorPlanID UNIQUEIDENTIFIER,
    @ComponentType NVARCHAR(50),
    @Properties NVARCHAR(MAX),
    @PositionX FLOAT,
    @PositionY FLOAT
AS
BEGIN
    INSERT INTO Customizations (CustomizationID, FloorPlanID, ComponentType, Properties, PositionX, PositionY)
    VALUES (NEWID(), @FloorPlanID, @ComponentType, @Properties, @PositionX, @PositionY);
END;

-- Read
CREATE PROCEDURE usp_Customization_Read
    @CustomizationID UNIQUEIDENTIFIER
AS
BEGIN
    SELECT * FROM Customizations WHERE CustomizationID = @CustomizationID;
END;

-- Update
CREATE PROCEDURE usp_Customization_Update
    @CustomizationID UNIQUEIDENTIFIER,
    @ComponentType NVARCHAR(50),
    @Properties NVARCHAR(MAX),
    @PositionX FLOAT,
    @PositionY FLOAT
AS
BEGIN
    UPDATE Customizations
    SET ComponentType = @ComponentType,
        Properties = @Properties,
        PositionX = @PositionX,
        PositionY = @PositionY
    WHERE CustomizationID = @CustomizationID;
END;

-- Delete
CREATE PROCEDURE usp_Customization_Delete
    @CustomizationID UNIQUEIDENTIFIER
AS
BEGIN
    DELETE FROM Customizations WHERE CustomizationID = @CustomizationID;
END;
```

---

### ✅ `LegacyDataAudit` CRUD

```sql
-- Create
CREATE PROCEDURE usp_LegacyDataAudit_Create
    @LegacySystemID UNIQUEIDENTIFIER,
    @DataType NVARCHAR(50),
    @SourceFileName NVARCHAR(100),
    @ImportedBy NVARCHAR(100),
    @Status NVARCHAR(20)
AS
BEGIN
    INSERT INTO LegacyDataAudit (AuditID, LegacySystemID, DataType, SourceFileName, ImportedBy, Status)
    VALUES (NEWID(), @LegacySystemID, @DataType, @SourceFileName, @ImportedBy, @Status);
END;

-- Read
CREATE PROCEDURE usp_LegacyDataAudit_Read
    @AuditID UNIQUEIDENTIFIER
AS
BEGIN
    SELECT * FROM LegacyDataAudit WHERE AuditID = @AuditID;
END;

-- Update
CREATE PROCEDURE usp_LegacyDataAudit_Update
    @AuditID UNIQUEIDENTIFIER,
    @Status NVARCHAR(20)
AS
BEGIN
    UPDATE LegacyDataAudit
    SET Status = @Status
    WHERE AuditID = @AuditID;
END;

-- Delete
CREATE PROCEDURE usp_LegacyDataAudit_Delete
    @AuditID UNIQUEIDENTIFIER
AS
BEGIN
    DELETE FROM LegacyDataAudit WHERE AuditID = @AuditID;
END;
```

---
Here’s a set of **sample JSON payloads** for each table in your solution: `Users`, `Projects`, `FloorPlans`, `Customizations`, and `LegacyDataAudit`.

These payloads simulate the data you might **receive from a frontend** or use for **API testing**, such as with Postman or a JavaScript frontend sending requests to your Azure Function App or App Service API.

---

### 👤 **Users Table**

```json
{
  "UserID": "bde02e77-147e-44f6-a7b5-15f81ec0c802",
  "Username": "jdoe",
  "Email": "jdoe@example.com",
  "Role": "Designer"
}
```

---

### 🗂️ **Projects Table**

```json
{
  "ProjectID": "e1b12d5e-378c-42ae-bacc-f0f83d84c71b",
  "UserID": "bde02e77-147e-44f6-a7b5-15f81ec0c802",
  "ProjectName": "Maple Townhouse Block A",
  "Description": "Initial design for townhouse units in Maple Grove"
}
```

---

### 🏠 **FloorPlans Table**

```json
{
  "FloorPlanID": "44f83f47-fb69-4b7f-9fcb-6e4edccae3f7",
  "ProjectID": "e1b12d5e-378c-42ae-bacc-f0f83d84c71b",
  "FloorPlanName": "2BHK North-Facing",
  "BaseFilePath": "https://yourstorage.blob.core.windows.net/floorplans/2bhk-north.svg",
  "ThumbnailUrl": "https://yourstorage.blob.core.windows.net/thumbnails/2bhk-north.png"
}
```

---

### 🎨 **Customizations Table**

```json
{
  "CustomizationID": "c556a7e8-8046-4c4c-bc9e-87259d6bd013",
  "FloorPlanID": "44f83f47-fb69-4b7f-9fcb-6e4edccae3f7",
  "ComponentType": "Wall",
  "Properties": {
    "color": "#cccccc",
    "thickness": "6in",
    "material": "gypsum"
  },
  "PositionX": 200.5,
  "PositionY": 118.75
}
```

> Store `Properties` as a JSON **string** in SQL. The above is its expanded object view.

Serialized format for SQL:

```json
"Properties": "{\"color\":\"#cccccc\",\"thickness\":\"6in\",\"material\":\"gypsum\"}"
```

---

### 📜 **LegacyDataAudit Table**

```json
{
  "AuditID": "ed3421e4-0e4b-432f-a03b-70db68b8829d",
  "LegacySystemID": "98324ab0-1d1e-43c8-a70e-8b19e7ed9f56",
  "DataType": "Project_Metadata",
  "SourceFileName": "vb6_project_data_2023.csv",
  "ImportedBy": "migration.service",
  "Status": "Imported"
}
```

---

Would you like a Postman collection for these samples, or an OpenAPI (Swagger) spec describing the API that handles these payloads?




Would you like a downloadable `.sql` file or a PowerShell script to execute and deploy these to your Azure SQL environment?

---

Would you like me to generate a `.sql` file for download containing all these procedures?


## ☁️ **Azure Services Involved**

### 🔹 Azure App Service (or Azure Function App)
- Hosts the backend REST API for the HTML5 frontend.
- Handles user actions, floor plan CRUD, customization submissions, etc.

### 🔹 Azure SQL Database
- Hosts all relational data like users, projects, customizations, audit records.

### 🔹 Azure Service Bus
- **Queue**: For background processing (e.g., saving customizations, triggering notifications).
- **Topics/Subscriptions**: To notify other microservices (e.g., analytics, user activity).

### 🔹 Azure Storage
- **Blob Storage**: Stores base floor plan images, custom layouts (SVG, PNG, etc).
- **Table Storage**: Optionally stores event logs, non-relational metadata.
- **File Share**: For legacy VB6 project files if needed during migration.

### 🔹 Azure Data Factory
- ETL jobs to migrate and transform VB6/MS Access/SQL Server legacy data into the new SQL schema.
- Batch historical data into `LegacyDataAudit` for tracking.

---


Set Up Steps 

Creating a serverless API using Azure that leverages Service Bus to communicate with an SQL Database involves several steps. Here's a high-level overview of how you can set this up:

1. **Set Up Azure SQL Database**:
   - Create an Azure SQL Database instance.
   - Set up the necessary tables and schemas you'll need for your application.

2. **Create Azure Service Bus**:
   - Set up an Azure Service Bus namespace.
   - Within the namespace, create a queue or topic (based on your requirement).

3. **Deploy Serverless API using Azure Functions**:
   - Create a new Azure Function App.
   - Develop an HTTP-triggered function that will act as your API endpoint.
   - In this function, when data is received, send a message to the Service Bus queue or topic.

4. **Deploy 2 Service Bus Triggered Function**:
   - Create another Azure Function that is triggered by the Service Bus queue or topic.
   - This function will read the message from the Service Bus and process it. The processing might involve parsing the message and inserting the data into the Azure SQL Database.

5. **Deploy a Timer Triggered Function**:
   - Create another Azure Function that is triggered when a file is dropped in a container.
   - This function will stream in a file, read it and place on the service bus topic.

6. **Implement Error Handling**:
   - Ensure that you have error handling in place. If there's a failure in processing the message and inserting it into the database, you might want to log the error or move the message to a dead-letter queue.

7. **Secure Your Functions**:
   - Ensure that your HTTP-triggered function (API endpoint) is secured, possibly using Azure Active Directory or function keys.

8. **Optimize & Monitor**:
   - Monitor the performance of your functions using Azure Monitor and Application Insights.
   - Optimize the performance, scalability, and cost by adjusting the function's plan (Consumption Plan, Premium Plan, etc.) and tweaking the configurations.

9. **Deployment**:
   - Deploy your functions to the Azure environment. You can use CI/CD pipelines using tools like Azure DevOps or GitHub Actions for automated deployments.

By following these steps, you'll have a serverless API in Azure that uses Service Bus as a mediator to process data and store it in an SQL Database. This architecture ensures decoupling between data ingestion and processing, adding a layer of resilience and scalability to your solution.


## Appplication Setting 

|Key|Value | Comment|
|:----|:----|:----|
|AzureWebJobsStorage|[CONNECTION STRING]|RECOMMENDATION :  store in AzureKey Vault.|
|ConfigurationPath| [CONFIGURATION FOLDER PATH] |Folder is optional
|ApiKeyName|[API KEY NAME]|Will be passed in the header  :  the file name of the config.
|AppName| [APPLICATION NAME]| This is the name of the Function App, used in log analytics|
|StorageAcctName|[STORAGE ACCOUNT NAME]|Example  "AzureWebJobsStorage"|
|ServiceBusConnectionString|[SERVICE BUS CONNECTION STRING]|Example  "ServiceBusConnectionString".  Recommmended to store in Key vault.|
|DatabaseConnection|[DATABASE CONNECTION STRING]|Example  "DatabaseConnection". Recommmended to store in Key vault.|
|TimerInterval|[TIMER_INTERVAL]|Example  "0 */1 * * * *" 1 MIN|


> **Note:**  Look at the configuration file in the **Config** Folder and created a Table to record information.

## Configuration Files 

> **Note:** The **Configuration** is located in the  FunctionApp  in a **Config** Folder.

|FileName|Description|
|:----|:----|
|0F52331916F24E3DA3CC64A74E52AC6E.json| Create a user |
|6347356F99554F82BF498ECCC3B14C58.json| Retrieve user information|
|F4A6650EFDF54314A46AFAF9D8F9C700.json| Update user information|
|00A490C5637E4634AA146E966C15AA71.json| Delete user information|

|0F52331916F24E3DA3CC64A74E52AC6E.json| Create a user |
|6347356F99554F82BF498ECCC3B14C58.json| Retrieve user information|
|F4A6650EFDF54314A46AFAF9D8F9C700.json| Update user information|
|00A490C5637E4634AA146E966C15AA71.json| Delete user information|


> Create the following blob containers and share in azure storage

|ContainerName|Description|
|:----|:----|
|config|Location for the configuration files|
|pickup|Thes are files that are copied from the SFTP share and dropped in the pickup container |
|processed|These are files the have been parsed and dropped in th processed container|

|Table|Description|
|:----|:----|
|csvbatchfiles|Track the CSV parsed files|
|training[YYYYMMDD]|N0 SQL DataStore|



## 📘 **Example Scenario: Floor Plan Customization Journey**

1. A user logs into the HTML5 application.
2. Selects a project and opens a floor plan from Blob Storage.
3. The app allows interactive features using JS events (click, drag, drop) to customize the plan.
4. Customizations are captured in JSON and sent via API.
5. API writes the customization:
    - Into SQL (`Customizations`)
    - Sends a message to **Azure Service Bus Queue** (`FloorPlanUpdatedEvent`)
6. Another Azure Function listens to this queue:
    - Generates updated thumbnails.
    - Logs activity to **Azure Table Storage**.
7. A reporting service pulls logs and data from SQL + Table Storage for analysis.

---

## 🧠 **Design Highlights**

- **Reusability & Extensibility**: By storing component properties as JSON, we support evolving design types.
- **Separation of Concerns**: Azure Service Bus decouples UI events from background processing.
- **Cloud-Native**: Using Azure SQL, Blob/Table storage, and Functions/Service Bus ensures scalability.
- **Historical Data Strategy**: Legacy audit tracking with Azure Data Factory and `LegacyDataAudit`.

---

