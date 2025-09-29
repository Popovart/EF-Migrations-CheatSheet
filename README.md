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

## List migrations

```bash
dotnet ef migrations list
```

- Lists migrations found in the target assembly in chronological order.

MAUI notes:
- If the `DbContext` is in a separate project, specify both the data project and the startup MAUI app:

```bash
dotnet ef migrations list --project Your.DataProject --startup-project Your.MauiApp
```

- If you have multiple contexts, specify one:

```bash
dotnet ef migrations list --context LocalDbContext
```

## Check for pending model changes

```bash
dotnet ef migrations has-pending-model-changes
```

- Compares the current EF model to the last migration snapshot.
- Returns a non-zero exit code if changes are detected (useful for CI checks).

MAUI notes:
- Typical usage in a multi-project solution:

```bash
dotnet ef migrations has-pending-model-changes --project Your.DataProject --startup-project Your.MauiApp --context LocalDbContext --no-build
```

## Update the database

```bash
dotnet ef database update
```

- Applies the latest migration to the configured database.
- To update to a specific migration (by name or id):

```bash
dotnet ef database update InitialCreate
```

MAUI notes:
- If the `DbContext` is in a separate project, specify both the data project and the startup MAUI app:

```bash
dotnet ef database update --project Your.DataProject --startup-project Your.MauiApp
```

- If you have multiple contexts, specify one:

```bash
dotnet ef database update --context LocalDbContext
```

- Speed up repeated runs by skipping build when nothing changed:

```bash
dotnet ef database update --no-build
```

- (Optional) Override connection string for the update:

```bash
dotnet ef database update --connection "Data Source=app.db"
```

- Roll back all migrations (to empty database schema):

```bash
dotnet ef database update 0
```

Note:
- `dotnet ef database update` is a manual way to apply migrations and an alternative to calling `Database.Migrate()` at runtime.
- If your MAUI app already calls `Database.Migrate()` on startup (e.g., in `MauiProgram`), migrations are applied automatically when the app runs.
- Use the CLI command for local development, CI, or when you need to apply migrations without launching the app.

## Apply migrations at runtime in MAUI (via DI, non-blocking UI)

Apply migrations asynchronously during app startup so the UI thread isn't blocked. This is convenient for local/dev builds. For production, prefer SQL scripts or migration bundles; applying migrations at runtime can cause issues (concurrency, permissions). See Microsoft docs: https://learn.microsoft.com/en-gb/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli

```csharp
builder.Services.AddDbContext<LocalDbContext>(
			options =>
			{
				var dbPath = Path.Combine(FileSystem.AppDataDirectory, "LocalDb.db");
				Directory.CreateDirectory(Path.GetDirectoryName(dbPath) ?? throw new InvalidOperationException("No directory in path"));
				options.UseSqlite($"Data Source={dbPath}");
				
				// For more configurations:
				// var cs = new SqliteConnectionStringBuilder
				// {
				//     DataSource = dbPath,
				//     Mode = SqliteOpenMode.ReadWriteCreate
				// }.ToString();
				// options.UseSqlite(cs);
			}
		);

var app = builder.Build();
		_ = Task.Run(async () =>
		{
			using var scope = app.Services.CreateScope();
			var db = scope.ServiceProvider.GetRequiredService<LocalDbContext>();
			await db.Database.MigrateAsync();
		});
```

Notes:
- Runs database migration off the UI thread to avoid startup freezes.
- Don't call `EnsureCreated()` before `MigrateAsync()`.
- For production environments, prefer `dotnet ef migrations script` or `dotnet ef migrations bundle`.
