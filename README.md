# Student API

Azure Functions API using .NET 10 and SQL Server. `GET /api/WhoAmI` returns `Bardia Bashiri` as plain text, without needing a database connection. The student endpoints remain under `/api/students`.

## Deploy through GitHub Actions

### 1. Create the Azure Function App

Open [Azure Portal](https://portal.azure.com), search for **Function App**, and select **Create**.

Choose these settings:

| Setting | Value |
| --- | --- |
| Hosting plan | Flex Consumption |
| Subscription | Your Azure subscription |
| Resource group | Create `student-api-rg` |
| Function App name | A unique name, for example `bardia-student-api-258` |
| Region | An available region near you |
| Runtime stack | .NET |
| Version | .NET 10, isolated worker |
| Operating system | Linux (Flex Consumption default) |

Keep the default storage and monitoring settings, leave public network access enabled, and select **Review + create**, then **Create**. Leave automatic GitHub deployment disabled during creation; this repository already includes the workflow.

Use Flex Consumption for this guide: the workflow includes `sku: flexconsumption`. .NET 10 is supported on Flex Consumption, but not on the older Linux Consumption plan. See [Microsoft's hosting instructions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-create-function-app-portal) and [.NET support](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#supported-versions).

### 2. Set the database connection

Open the new Function App, then **Settings > Environment variables > App settings**. Add:

| Name | Value |
| --- | --- |
| `DATABASE_CONNECTION_STRING` | Paste the entire connection string supplied in the assignment |

Select **Apply/Save** and confirm any restart prompt. Use **App settings**, because the existing code reads this value as an environment variable.

The supplied connection targets `goodbooks-db-server.database.windows.net`, database `azure-function`. You do not need to create another database. Keep the full connection string in Azure, outside source control. Azure provisions the host storage and runtime configuration when you create the app. See [Azure app settings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-app-settings).

### 3. Download the deployment credentials

In the Function App, open **Settings > Configuration** and enable **SCM Basic Auth Publishing Credentials**, then save. It may appear under **General settings**.

Return to **Overview**, select **Get publish profile** (or **Download publish profile**), and open the downloaded file in a text editor. You will paste its entire XML contents into GitHub in the next step. Keep the downloaded file outside the repository.

This simple setup uses publish profile authentication. Microsoft recommends OIDC for production; publish profiles require a stored deployment secret and SCM basic authentication. See the [official publish profile setup](https://github.com/Azure/functions-action/blob/master/docs/authentication.md#publish-profile).

### 4. Configure GitHub

Open [your repository settings](https://github.com/Bardia258/csharp-lab-serverless-func/settings/secrets/actions), then **Secrets and variables > Actions**.

On the **Variables** tab, select **New repository variable**:

| Name | Value |
| --- | --- |
| `AZURE_FUNCTIONAPP_NAME` | The exact Function App name from step 1, without a URL |

On the **Secrets** tab, select **New repository secret**:

| Name | Value |
| --- | --- |
| `AZURE_FUNCTIONAPP_PUBLISH_PROFILE` | The complete XML contents of the downloaded publish profile |

The database connection string belongs in Azure; GitHub only needs the deployment credentials.

### 5. Push the changes

From the repository root, run:

```powershell
git add .github/workflows/deploy.yml .gitignore README.md ServerlessStudents/WhoAmI.cs ServerlessStudents/ServerlessStudents.csproj
git commit -m "Add WhoAmI endpoint and Azure deployment workflow"
git push origin main
```

Open the repository's **Actions** tab and select **Deploy Azure Functions** to follow the deployment. The workflow installs .NET 10, runs `dotnet publish` to restore dependencies and build the package, and deploys that package to Azure. It has no test step.

Every later push to `main` repeats the deployment automatically. To deploy again without a commit, choose **Actions > Deploy Azure Functions > Run workflow**.

### 6. Use the hosted API

Copy the Function App's **Default domain** from its Azure Overview page. Your endpoints are:

```text
https://<default-domain>/api/WhoAmI
https://<default-domain>/api/students
```

The first endpoint returns `Bardia Bashiri`. Use the actual default domain Azure shows; it may include a generated suffix.

If student requests fail with a SQL firewall error, the database owner/instructor must allow the Function App's outbound IP addresses on the existing SQL server. If deployment returns `401`, check SCM basic authentication and the GitHub publish profile secret. See [SQL firewall configuration](https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure?view=azuresql).

No local build, application run, or tests were performed as part of these changes.
