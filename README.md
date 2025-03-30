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

Here’s the **Azure architecture diagram** visualizing the flow from the HTML5 UI to all Azure services involved in the re-platformed interactive housing floor plan solution.

Now here’s the **Mermaid diagram** for the same:

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

Would you like me to generate this as a downloadable PNG or include sample Azure Bicep or Terraform for provisioning these resources too?
