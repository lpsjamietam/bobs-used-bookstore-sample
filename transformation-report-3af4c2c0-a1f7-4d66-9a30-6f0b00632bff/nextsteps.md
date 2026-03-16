# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all projects after transformation:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the migration to cross-platform .NET was completed without introducing any build-time issues. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the solution root to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Ensure there are no warnings that could indicate deprecated APIs or compatibility issues that may surface at runtime.

---

## 2. Run the Unit Tests

Execute the test project to verify that existing domain logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any failing tests
- Any skipped tests that may have been conditionally excluded during migration
- Code coverage gaps in critical domain logic

---

## 3. Validate the Data Layer

The `Bookstore.Data` project likely contains database access logic such as Entity Framework Core migrations or a data context. Perform the following checks:

- Confirm the correct database provider package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql`, or `Microsoft.EntityFrameworkCore.Sqlite`).
- If using Entity Framework Core, verify that migrations are up to date:

```bash
dotnet ef migrations list --project app/Bookstore.Data
dotnet ef database update --project app/Bookstore.Data
```

- Run any integration tests or manual checks against a local database instance to confirm data access works correctly.

---

## 4. Validate the Web Application

Run the `Bookstore.Web` project locally to verify runtime behavior:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- Application starts without runtime exceptions.
- All routes and pages load as expected.
- Authentication and authorization flows work correctly if applicable.
- Static assets are served properly.
- Any configuration values in `appsettings.json` are correct for the target environment.

---

## 5. Validate the CDK Project

The `Bookstore.Cdk` project appears to define infrastructure. Verify the following:

- All AWS CDK or infrastructure dependencies are correctly referenced and compatible with the current .NET version.
- Synthesize the CDK stack to confirm it produces valid output:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Or if using the CDK CLI:

```bash
cdk synth
```

Review the synthesized output for correctness before any deployment.

---

## 6. Check for Runtime Compatibility Issues

Even with a clean build, certain areas require manual review:

- **Reflection-based code**: Ensure any use of reflection is compatible with the target .NET runtime.
- **Platform-specific APIs**: Search for any remaining usage of Windows-only APIs (e.g., `System.Drawing`, registry access, COM interop) that may cause runtime failures on non-Windows platforms.
- **Configuration and environment variables**: Confirm that `IConfiguration` sources are set up correctly for the new hosting model if the project moved to a minimal hosting API.
- **NuGet package versions**: Review `Bookstore.Data` and `Bookstore.Domain` for any packages that may have been updated to versions with breaking changes during transformation.

---

## 7. Publish the Application

Once all validation steps pass, publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Review the contents of the `./publish` directory to confirm all required files, configuration, and assets are present before deploying to the target environment.