# NASA News Feed Processor

A Java desktop application that fetches and manages live news articles from NASA's public RSS feed. The app stores articles in a local SQL Server database and provides a GUI for browsing, editing, uploading, and administering content — with role-based access control.

---

## Overview

The application connects to the **NASA News Release RSS feed** (`https://www.nasa.gov/news-release/feed/`), parses incoming articles using a StAX XML parser, and persists them in a relational database. A Swing-based desktop GUI lets users view, edit, and upload articles, while admins can manage user accounts.

**Key features:**
- Live RSS ingestion from NASA's official news feed
- Parses article title, link, description, publication date, and cover image
- Downloads and stores article images locally
- Full CRUD operations on articles via GUI
- Role-based access: standard users vs. admins
- User authentication with bcrypt-hashed passwords
- Admin panel for user account management

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Java |
| Build | Maven (multi-module: `ArticleManager`, `Dao`, `Utilities`) |
| GUI | Java Swing (NetBeans `.form` files) |
| Data access | Custom DAO layer (`Repository` interface + SQL Server impl) |
| XML parsing | StAX (`javax.xml.stream`) |
| Database | Microsoft SQL Server |
| Auth | bcrypt password hashing |
| Data binding | JAXB |

---

## Project Structure

```
Nasa-space-data-pipeline/
├── pom.xml                        # Parent POM — aggregates all modules
├── Database.sql                   # SQL Server schema + stored procedures
│
├── ArticleManager/                # Main application module
│   └── src/main/java/hr/algebra/
│       ├── ArticleManager.java    # Entry point: login → main window
│       ├── parsers/rss/
│       │   └── ArticleParser.java # StAX RSS parser for NASA feed
│       └── view/
│           ├── LoginPanel / LoginDialog       # Authentication
│           ├── EditArticlesPanel              # Browse & edit stored articles
│           ├── UploadArticlesPanel            # Fetch & upload new articles
│           ├── AdminPanel                     # Admin-only article management
│           ├── AddUserDialog                  # Admin-only user creation
│           └── ImageDropFrame                 # Drag-and-drop image handler
│
├── Dao/                           # Data access layer
│   └── src/main/java/hr/algebra/
│       ├── dal/
│       │   ├── Repository.java             # CRUD interface
│       │   ├── RepositoryFactory.java      # Config-driven factory
│       │   └── sql/
│       │       ├── SqlRepository.java      # SQL Server implementation
│       │       └── DataSourceSingleton.java
│       └── model/
│           ├── Article.java                # Article entity
│           └── NewsFeedUser.java           # User entity
│
└── Utilities/                     # Shared utilities
    └── src/main/java/hr/algebra/
        ├── factory/
        │   ├── ParserFactory.java          # Returns correct XML parser
        │   └── UrlConnectionFactory.java   # Opens HTTP connections
        └── utilities/
            ├── FileUtils.java
            ├── IconUtils.java
            ├── ImageTransferHandler.java   # Drag-and-drop support
            ├── JAXBUtils.java
            └── MessageUtils.java
```

---

## Database Schema

The SQL schema (`Database.sql`) defines two tables and a full set of stored procedures.

### Tables

**`Article`** — stores fetched news items  
`IDArticle` (PK) · `Title` · `Link` · `Description` · `PicturePath` · `PublishedDate`

**`NewsFeedUser`** — application accounts  
`UserId` (PK) · `Username` (unique) · `PasswordHash` (bcrypt) · `IsAdmin`

### Stored Procedures

| Procedure | Description |
|-----------|-------------|
| `createArticle` | Insert new article, returns new ID |
| `updateArticle` | Update existing article by ID |
| `deleteArticle` | Delete one article by ID |
| `deleteArticles` | Delete all articles |
| `selectArticle` | Get single article by ID |
| `selectArticles` | Get all articles |
| `createUser` | Insert new user, returns new ID |
| `selectUser` | Lookup user by username |

---

## Setup

### Prerequisites

- Java (JDK 8+)
- Apache Maven
- Microsoft SQL Server (local or remote instance)

### 1. Database

Run `Database.sql` against your SQL Server instance to create the `ARTICLES` database, tables, and stored procedures.

### 2. Configuration

Edit `Dao/src/main/resources/config/db.properties` with your SQL Server connection details:

```properties
db.url=jdbc:sqlserver://localhost;databaseName=ARTICLES
db.username=sa1
db.password=<your_password>
```

### 3. Build

```bash
mvn clean install
```

### 4. Run

```bash
java -jar ArticleManager/target/ArticleManager-1.0-SNAPSHOT.jar
```

A login dialog will appear. Use the seeded `admin` account (configured in `Database.sql`) or create additional accounts through the Admin panel.

---

## How It Works

1. **Login** — the app opens a login dialog; credentials are verified against bcrypt hashes in the database.
2. **Upload tab** — triggers `ArticleParser`, which opens an HTTP connection to `https://www.nasa.gov/news-release/feed/`, parses the RSS XML using StAX, downloads cover images locally, and persists all articles through the DAO layer.
3. **Edit tab** — loads stored articles from the database; users can update fields or delete entries.
4. **Admin tab** — visible to admin accounts only; provides full article management and user creation via `AddUserDialog`.

---

## Data Source

This project consumes the **NASA News Release RSS feed**:  
`https://www.nasa.gov/news-release/feed/`

NASA's open data is provided under their [media usage guidelines](https://www.nasa.gov/multimedia/guidelines/index.html).

---

## Notes

- Article images are downloaded to a local `assets/` directory on first fetch; paths are stored in the database.
- The `predict.java` / live-demo scripts in the repo tree are build artefacts from the Maven compile, not additional source files.
- The `.form` files are NetBeans GUI designer descriptors — the actual logic is in the corresponding `.java` files.
