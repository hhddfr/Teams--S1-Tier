## Bot Service Infrastructure & Configuration Guide

This repository contains a Microsoft Teams bot built using the **Bot Framework SDK** and **Teams Toolkit**. It is designed with "Configuration as Code", using automated workflows to deploy a secure, enterprise-ready environment on Microsoft Azure.

---

### 📂 Project Architecture & File Roles

The project is organized into three distinct layers: application logic, automation orchestration, and infrastructure-as-code.

#### Application Logic (Node.js)
These files define the bot's "brain" and how it communicates with users.
* **`index.js`**: The central engine. It initializes the `CloudAdapter` to process messages and starts a **Restify server** (defaulting to port **3978**) to listen for incoming activities.
* **`teamsBot.js`**: The core logic provider. It utilizes the `@microsoft/agents-hosting` library to manage bot behaviors, handle state in memory, and process file downloads.
* **`config.js`**: A clean configuration wrapper that maps environment variables into the application, ensuring sensitive IDs are handled dynamically.
* **`package.json`**: Defines the project metadata, dependencies (like `botbuilder`), and scripts for both local development and production execution.

#### Orchestration (Teams Toolkit YAML)
These files act as the project's "instruction manual," telling the Teams Toolkit how to build and move the code.
* **`teamsapp.yml`**: The production lifecycle manager. It automates the creation of the Teams app identity, runs the Azure deployment scripts, and handles the final publishing to the Teams Admin Center.
* **`teamsapp.local.yml`**: Tailored for the "inner-dev loop." It configures the local environment and sets up the bot registration so you can test changes instantly without a full cloud deployment.

#### Infrastructure (Azure Bicep)
Bicep files are used to define the exact Azure hardware and security policies required for a production-grade bot.
* **`azure.bicep`**: The primary infrastructure template. It provisions:
    * **Azure App Service**: The hosting environment for the Node.js code, enforced with **HTTPS only** and **TLS 1.2**.
    * **Azure Key Vault**: A secure hardware security module used to store the `BOT_PASSWORD` secret, ensuring it never appears in plain-text logs or settings.
    * **Networking Rules**: A strict firewall that **denies all** general internet traffic, allowing only the official **AzureBotService** to reach the bot's endpoint.
    * **Monitoring**: Integrated **Application Insights** and **Log Analytics** for real-time error tracking and performance metrics.
* **`azurebot.bicep`**: A specialized module that registers the bot with the **Azure Bot Service**. It connects the code to the **Microsoft Teams** channel and defines the communication endpoint.
* **`azure.parameters.json`**: A configuration file that feeds specific values (like the **B1 webAppSKU**) into the Bicep templates without modifying the underlying infrastructure code.

### 🚀 Deployment Lifecycle

The deployment is a fully automated, three-phase process managed by the Teams Toolkit.

#### 1. Provisioning (Infrastructure Setup)
This phase builds the "house" for your code to live in:
* **Identity**: Creates an Azure Active Directory (AAD) application to serve as the bot's unique identity.
* **Resource Creation**: Executes the Bicep files to stand up the App Service and Bot Registration.
* **Secret Security**: The `BOT_PASSWORD` is automatically stored in an **Azure Key Vault**. The Web App is granted access via a **System-Assigned Managed Identity**, ensuring the secret is never stored in plain text within app settings.

#### 2. Deployment (Code Upload)
This phase moves your code into the "house":
* **Build**: Runs `npm install --production` to package only the necessary files.
* **Zip Deploy**: Packages the project and uploads it directly to the **Azure App Service**.
* **Configuration**: Injects the `BOT_ID` and a Key Vault reference for the password into the Web App's environment variables.

#### 3. Publishing (Teams Integration)
This final phase connects the bot to the Teams client:
* **Validation**: The toolkit validates the manifest and app package against Microsoft's official schemas.
* **Admin Submission**: The package is uploaded to the **Teams Admin Center**, allowing organization-wide approval and installation.

### 🛡️ Security & Networking
The infrastructure is pre-configured with industry-standard security:
* **IP Security Restrictions**: The Web App is configured to **Deny all** traffic by default. It only accepts requests from the **AzureBotService** service tag, preventing unauthorized public access.
* **Encryption**: Uses a Key Vault encryption key for application-level data protection.
* **HTTPS Only**: Enforces **TLS 1.2** and redirects all traffic to secure HTTPS endpoints.
