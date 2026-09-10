# Security and Identity Patterns

## SECURITY-001: Never Store Secrets in Code or Configuration Files

**Impact**: CRITICAL (Prevents credential leaks and security breaches)

Hardcoding secrets or committing them to source control exposes credentials to anyone with repository access, leading to potential data breaches and unauthorized access.

### Incorrect

```csharp
// ❌ BAD: Hardcoded secrets in code
public class EmailService
{
    private const string ApiKey = "sk_live_123456789"; // Exposed in source!
    private const string ConnectionString =
        "Server=prod;Password=P@ssw0rd!"; // Security breach!
}

// ❌ BAD: In appsettings.json committed to git
{
  "ConnectionStrings": {
    "Default": "Server=prod;User=admin;Password=secret123"
  },
  "EmailSettings": {
    "ApiKey": "sk_live_123456789"
  }
}
```

### Correct

```csharp
// ✅ GOOD: Use User Secrets for local development
// Command: dotnet user-secrets set "EmailSettings:ApiKey" "sk_live_123"
// Command: dotnet user-secrets set "ConnectionStrings:Default" "Server=..."

// ✅ GOOD: Use Azure Key Vault or similar for production
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    var keyVaultUrl = builder.Configuration["KeyVaultUrl"];
    builder.Configuration.AddAzureKeyVault(
        new Uri(keyVaultUrl),
        new DefaultAzureCredential());
}

// ✅ GOOD: Access through strongly-typed configuration
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));

public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> settings)
    {
        _settings = settings.Value;
        // Use _settings.ApiKey safely
    }
}
```

### Additional Context

For local development, use:
- User Secrets (`dotnet user-secrets`) - stored outside project directory
- Environment variables - never committed to source control

For production, use:
- Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault
- Managed Identity for authentication (no credentials needed)
- Key rotation policies and audit logging

Never commit appsettings.Production.json with real secrets. Use appsettings templates instead.

### Reference

- [Safe Storage of App Secrets](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets)
- [Azure Key Vault Configuration Provider](https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration)
- [Environment Variables in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
