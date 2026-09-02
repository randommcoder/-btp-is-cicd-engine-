# SAP BTP Integration Suite - CI/CD Transport Engine

This repository serves as a centralized engine for automating imports, transports, and rollbacks for SAP Integration Suite artifacts using GitHub Actions. 

## Architecture & Concept

To keep your deployment logic up-to-date and separated from your integration source code, this architecture uses a **Two-Repository Strategy**:

1. **The Engine Repository (This Repo):** You fork this repository into your organization. It contains all the complex GitHub Actions, scripts, and API interactions with SAP BTP. By keeping this as a fork, you can regularly sync it to receive updates and bug fixes without disrupting your code. **Do not store your CPI packages here.**
2. **The Source Code Repository:** You create a second, separate repository in your organization. This repository will exclusively hold your SAP CPI packages, iFlows, Value Mappings, and configuration files.

### Why this approach?
If you simply download the workflow files and put them directly into your source code repository, you will be disconnected from future updates. By maintaining a fork of the engine, you can continuously sync upstream changes while keeping your actual CPI code safely isolated in its own repository.

---

## Initial Setup & Environment Configuration

### Step 1: Fork the Engine
Fork this repository into your GitHub organization (e.g., `your-org/-btp-is-cicd-engine-`).

### Step 2: Create the Source Code Repository
Create a new, empty repository in your organization (e.g., `your-org/cpi-source-code`). This is where the actual integration packages will be stored.

### Step 3: Configure GitHub Environments
In your **Source Code Repository**, go to **Settings > Environments** and create environments matching your landscape. For example:
- `DEV-ENV`
- `TEST-ENV`
- `PROD-ENV`

### Step 4: Configure Secrets and Variables
For **each environment** you created in Step 3, you need to configure the following Environment Secrets and Variables to allow GitHub to communicate with your SAP BTP subaccounts.

#### Environment Variables (Non-sensitive)
- `BTP_API_URL`: The API URL for your Integration Suite (e.g., `https://<subaccount>.it-cpioem.cfapps.eu10.hana.ondemand.com`)
- `BTP_TOKEN_URL`: The OAuth Token URL (e.g., `https://<subaccount>.authentication.eu10.hana.ondemand.com/oauth/token`)
- `BTP_API_USER`: The Client ID of your BTP Process Integration Service Key.

#### Environment Secrets (Sensitive)
- `BTP_API_PASSWORD`: The Client Secret of your BTP Process Integration Service Key.

*(Note: Depending on your exact BTP setup, if you have separate technical users for deployment, you may also need `BTP_TEC_USER` and `BTP_TEC_PASSWORD` variables/secrets as defined in your BTP Service Keys).*

---

## How to Run the Workflows (It's extremely easy!)

We've designed this so you get the standard GitHub Actions UI with all the text boxes and dropdowns, without having to write any code!

Since the engine and the source code are in separate repositories, you will run the workflows from your **Source Code Repository** by using the pre-built "Caller Templates". 

### Setting up Caller Workflows
1. In your **Source Code Repository**, create a folder named `.github/workflows/`.
2. Go to the `Caller-Templates/` folder in this repository.
3. Copy all the `.yml` files from `Caller-Templates/` and paste them into your Source Code repository's `.github/workflows/` folder.
4. **Important**: Open each copied file and change `your-org/-btp-is-cicd-engine-` to match your actual organization and repository name where you forked the Engine.

### Running a Transport or Import
That's it! Because of the way these templates are structured, you simply:
1. Go to your **Source Code Repository** on GitHub.
2. Click the **Actions** tab at the top.
3. Select the workflow you want to run (e.g., "[Caller] [ADMIN] Transport and Deploy as Package").
4. Click the **Run workflow** button on the right side.
5. A nice UI form will pop up asking for the Environment, Package Name, Branch, etc. Fill in the boxes and hit Run!

Your Source Code repository remains incredibly clean, containing only your integration artifacts and these lightweight UI templates. All the heavy lifting is completely managed by the Engine repository in the background.

---

## Managing Environment Parameters (Zero-Touch Transports)

The engine automatically manages all environment-specific configurations (like endpoint URLs, proxy ports, and credentials) so you never have to manually update them after a transport.

### The `Transport_Parameters.json` File
To use this feature, create a file named `Transport_Parameters.json` in the root of your **Source Code Repository** on each of your environment branches (e.g., your QA branch and your PRD branch). This file acts as the configuration dictionary for that specific environment.

Example format (for your QA branch):
```json
{
    "SF_Endpoint_URL": "https://api.successfactors.eu/qa",
    "Email_Receiver_Address": "qa-team@company.com"
}
```

### How it Works
When you trigger a transport (e.g., from DEV to QA), the engine checks out your target branch (QA) and reads its specific `Transport_Parameters.json` file. It dynamically scans your package for `parameters.prop` files and `ConfigParams_INTEGRATION_FLOW.json`. 

It matches the Parameter Keys in your package against the keys in your JSON file and **automatically injects the correct values for the target environment** right before deploying to SAP Integration Suite!
