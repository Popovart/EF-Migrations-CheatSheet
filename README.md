## Add a new migration

```bash
dotnet ef migrations add InitialCreate -o Migrations
```

- Creates a migration named `InitialCreate`.
- `-o Migrations` writes files to the `Migrations` folder of the target project.

MAUI notes:
- If the `DbContext` lives in a different project than the MAUI app, specify both the data project and the startup MAUI project:

```bash
dotnet ef migrations add InitialCreate -o Migrations --project Your.DataProject --startup-project Your.MauiApp
```

- If you have multiple contexts, specify one:

```bash
dotnet ef migrations add InitialCreate -o Migrations --context LocalDbContext
```

Prerequisites:
- Install/update EF CLI: `dotnet tool update -g dotnet-ef` (or `dotnet tool install -g dotnet-ef`)
- Ensure design-time creation of the context works (e.g., implement `IDesignTimeDbContextFactory<TContext>` or configure the app host so EF can construct your `DbContext`).
