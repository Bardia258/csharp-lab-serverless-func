# Student API

Azure Functions API using .NET 10 and SQL Server.

- `GET /api/WhoAmI` returns `Bardia Bashiri` as plain text.
- Student endpoints are under `/api/students`.

## Azure deployment

The Azure Function App `student-api` is connected to this GitHub repository through Azure Deployment Center. Azure generated `.github/workflows/main_student-api.yml` and configured its deployment identity and GitHub secrets.

Every push to `main` builds and deploys the app automatically. The workflow uses .NET 10, publishes the project from `ServerlessStudents`, and deploys the output to Azure. There is no test step.

To set this up again:

1. Create a Function App using .NET 10 isolated worker and Flex Consumption.
2. Open **Deployment > Deployment Center**, select GitHub, and authorize your account.
3. Select `Bardia258/csharp-lab-serverless-func`, branch `main`.
4. Choose **Add a workflow** and the suggested new **User-assigned identity**, then save. Keep only one deployment workflow enabled.
5. Ensure the generated workflow builds from `ServerlessStudents` using .NET 10 and `dotnet publish`.

See [Microsoft's Deployment Center instructions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-how-to-github-actions#create-the-workflow-configuration-in-the-portal).

## Database setting

In the Function App, open **Settings > Environment variables > App settings**. Add `DATABASE_CONNECTION_STRING` with the entire connection string supplied in the assignment, then apply/save.

The existing code reads this environment variable. The connection targets `goodbooks-db-server.database.windows.net`, database `azure-function`. Keep the full connection string in Azure, outside source control.

If student requests fail with a SQL firewall error, ask the database owner/instructor to allow the Function App's outbound IP addresses on the SQL server.

## Push updates

Commit your changes, then run:

```powershell
git push origin main
```

Follow the deployment in the repository's **Actions** tab. You can also select the deployment workflow and **Run workflow** to deploy again without a new commit.

## Hosted URLs

Copy **Default domain** from the Function App's Azure Overview page and append `/api/WhoAmI` or `/api/students`. Use the actual domain shown by Azure, including any generated suffix.

No local build, application run, or tests were performed as part of these changes.
