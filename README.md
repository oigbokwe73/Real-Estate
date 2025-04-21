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

## 🗃️ **SQL Database Tables Design**

### 1. **Users**
```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY IDENTITY,
    Username NVARCHAR(50) NOT NULL,
    Email NVARCHAR(100) UNIQUE NOT NULL,
    Role NVARCHAR(20), -- Admin, Designer, Customer
    CreatedAt DATETIME DEFAULT GETDATE()
);
```

### 2. **Projects**
```sql
CREATE TABLE Projects (
    ProjectID INT PRIMARY KEY IDENTITY,
    UserID INT FOREIGN KEY REFERENCES Users(UserID),
    ProjectName NVARCHAR(100),
    Description NVARCHAR(500),
    CreatedAt DATETIME DEFAULT GETDATE(),
    LastModified DATETIME
);
```

### 3. **FloorPlans**
```sql
CREATE TABLE FloorPlans (
    FloorPlanID INT PRIMARY KEY IDENTITY,
    ProjectID INT FOREIGN KEY REFERENCES Projects(ProjectID),
    FloorPlanName NVARCHAR(100),
    BaseFilePath NVARCHAR(200),
    ThumbnailUrl NVARCHAR(200),
    CreatedAt DATETIME DEFAULT GETDATE()
);
```

### 4. **Customizations**
```sql
CREATE TABLE Customizations (
    CustomizationID INT PRIMARY KEY IDENTITY,
    FloorPlanID INT FOREIGN KEY REFERENCES FloorPlans(FloorPlanID),
    ComponentType NVARCHAR(50), -- e.g. Wall, Window, Door
    Properties NVARCHAR(MAX),   -- JSON string storing properties like size, color, orientation
    PositionX FLOAT,
    PositionY FLOAT,
    CreatedAt DATETIME DEFAULT GETDATE()
);
```

### 5. **LegacyDataAudit**
```sql
CREATE TABLE LegacyDataAudit (
    AuditID INT PRIMARY KEY IDENTITY,
    LegacySystemID INT,
    DataType NVARCHAR(50),
    SourceFileName NVARCHAR(100),
    ImportedBy NVARCHAR(100),
    ImportDate DATETIME,
    Status NVARCHAR(20) -- Imported, Failed, Validated
);
```

---

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
|99F77BEF300E4660A63A939ADD0BCF68.json| **Upload File** Parse CSV file --> Write Batched Files To Storage|
|43EFE991E8614CFB9EDECF1B0FDED37A.json| **File Parser** Parse CSV file --> File received from SFTP will use this process to parse files|
|43EFE991E8614CFB9EDECF1B0FDED37D.json| **Upload File** Parse JSON/CSV Directly to NO SQL DB|
|43EFE991E8614CFB9EDECF1B0FDED37C.json| **Service Bus Trigger for SQL DB** | Receive JSON payload and insert into SQL DB|
|43EFE991E8614CFB9EDECF1B0FDED37F.json| **Service Bus Trigger for No SQL DB** | Receive JSON payload and insert into NO SQL DB|
|43EFE991E8614CFB9EDECF1B0FDED37E.json| **Blob Trigger** Send parsed/sharded file  to Send to Service Bus|
|43EFE991E8614CFB9EDECF1B0FDED37B.json| **Search Resullt from NO SQLDB** |
|43EFE991E8614CFB9EDECF1B0FDED37G.json| **Search SQL DB. Return resultset** |
|3FB620B0E0FD4E8F93C9E4D839D00E1E.json| **Copy File from SFTP into the pickup folder** |
|3FB620B0E0FD4E8F93C9E4D839D00E1F.json| **Create a new Record in NoSQL Database** |
|CC244934898F46789734A9437B6F76CA.json| Encode Payload Request |
|6B427917E36A4DA281D57F9A64AD9D55.json| Get reports from DB  |


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

