# 🚀 WOWSQL PHP SDK

Official PHP client for [WOWSQL](https://wowsql.com) - MySQL Backend-as-a-Service with S3 Storage.

[![Packagist Version](https://img.shields.io/packagist/v/wowsql/wowsql-sdk.svg)](https://packagist.org/packages/wowsql/wowsql-sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## ✨ Features

### Database Features
- 🗄️ Full CRUD operations (Create, Read, Update, Delete)
- 🔍 Advanced filtering (eq, neq, gt, gte, lt, lte, like, isNull)
- 📄 Pagination (limit, offset)
- 📊 Sorting (orderBy)
- 🎯 Fluent query builder API
- 📝 Raw SQL queries (read-only)
- 📋 Table schema introspection
- 🔒 Built-in error handling

### Storage Features
- 📦 S3-compatible storage client
- ⬆️ File upload with automatic quota validation
- ⬇️ File download (presigned URLs)
- 📂 File listing with metadata
- 🗑️ File deletion
- 📊 Storage quota management
- 🌍 Multi-region support

## 📦 Installation

### Composer

```bash
composer require wowsql/wowsql-sdk
```

Or add to your `composer.json`:

```json
{
    "require": {
        "wowsql/wowsql-sdk": "^1.0.0"
    }
}
```

## 🚀 Quick Start

### Database Operations

```php
<?php
require 'vendor/autoload.php';

use WOWSQL\WOWSQLClient;
use WOWSQL\WOWSQLException;

// Initialize client
$client = new WOWSQLClient(
    'https://your-project.wowsql.com',
    'your-api-key'  // Get from dashboard
);

// Select data
$response = $client->table('users')
    ->select('id', 'name', 'email')
    ->eq('status', 'active')
    ->limit(10)
    ->get();

foreach ($response['data'] as $user) {
    echo $user['name'] . ' (' . $user['email'] . ')' . PHP_EOL;
}

// Insert data
$newUser = [
    'name' => 'Jane Doe',
    'email' => 'jane@example.com',
    'age' => 25
];

$result = $client->table('users')->create($newUser);
echo "Created user with ID: " . $result['id'] . PHP_EOL;

// Update data
$updates = ['name' => 'Jane Smith'];
$client->table('users')->update($result['id'], $updates);

// Delete data
$client->table('users')->delete($result['id']);
```

### Storage Operations

```php
<?php
use WOWSQL\WOWSQLStorage;
use WOWSQL\StorageException;

// Initialize storage client
$storage = new WOWSQLStorage(
    'your-project-slug',
    'your-api-key'
);

// Upload file
$storage->uploadFromPath('local-file.pdf', 'uploads/document.pdf', 'documents');

// Get presigned URL
$urlData = $storage->getFileUrl('uploads/document.pdf', 3600);
echo "File URL: " . $urlData['file_url'] . PHP_EOL;

// List files
$files = $storage->listFiles('uploads/');
foreach ($files as $file) {
    echo $file['key'] . ': ' . ($file['size'] / 1024 / 1024) . ' MB' . PHP_EOL;
}

// Delete file
$storage->deleteFile('uploads/document.pdf');

// Check storage quota
$quota = $storage->getQuota();
echo "Used: " . $quota['used_gb'] . " GB" . PHP_EOL;
echo "Available: " . $quota['available_gb'] . " GB" . PHP_EOL;
```

## 📚 Usage Examples

### Select Queries

```php
// Select all columns
$users = $client->table('users')->select('*')->get();

// Select specific columns
$users = $client->table('users')
    ->select('id', 'name', 'email')
    ->get();

// With filters
$activeUsers = $client->table('users')
    ->select('id', 'name', 'email')
    ->eq('status', 'active')
    ->gt('age', 18)
    ->get();

// With ordering
$recentUsers = $client->table('users')
    ->select('*')
    ->orderBy('created_at', true)  // desc = true
    ->limit(10)
    ->get();

// With pagination
$page1 = $client->table('users')
    ->select('*')
    ->limit(20)
    ->offset(0)
    ->get();

$page2 = $client->table('users')
    ->select('*')
    ->limit(20)
    ->offset(20)
    ->get();

// Pattern matching
$gmailUsers = $client->table('users')
    ->select('*')
    ->like('email', '%@gmail.com')
    ->get();

// Get single record by ID
$user = $client->table('users')->getById(123);
```

### Insert Data

```php
// Insert single record
$newUser = [
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'age' => 30,
    'status' => 'active'
];

$result = $client->table('users')->create($newUser);
echo "New user ID: " . $result['id'] . PHP_EOL;
```

### Update Data

```php
// Update by ID
$updates = [
    'name' => 'Jane Smith',
    'age' => 26
];

$result = $client->table('users')->update(1, $updates);
echo "Updated " . $result['affected_rows'] . " row(s)" . PHP_EOL;

// Update with conditions (using query builder)
$updated = $client->table('users')
    ->eq('status', 'inactive')
    ->update(['status' => 'active']);
```

### Delete Data

```php
// Delete by ID
$result = $client->table('users')->delete(1);
echo "Deleted " . $result['affected_rows'] . " row(s)" . PHP_EOL;

// Delete with conditions
$deleted = $client->table('users')
    ->eq('status', 'deleted')
    ->delete();
```

### Filter Operators

```php
// Equal
->eq('status', 'active')

// Not equal
->neq('status', 'deleted')

// Greater than
->gt('age', 18)

// Greater than or equal
->gte('age', 18)

// Less than
->lt('age', 65)

// Less than or equal
->lte('age', 65)

// Pattern matching (SQL LIKE)
->like('email', '%@gmail.com')

// Is null
->isNull('deleted_at')

// Is not null
->isNotNull('email')
```

### Advanced Queries

```php
// Multiple filters with sorting
$activeUsers = $client->table('users')
    ->select('id', 'name', 'email')
    ->eq('status', 'active')
    ->gt('age', 18)
    ->orderBy('created_at', true)  // desc
    ->limit(20)
    ->get();

// Complex filtering
$results = $client->table('products')
    ->select('*')
    ->gt('price', 100)
    ->lt('price', 1000)
    ->like('name', '%widget%')
    ->isNotNull('category')
    ->orderBy('price', false)  // asc
    ->limit(50)
    ->offset(0)
    ->get();
```

### Storage Operations

```php
$storage = new WOWSQLStorage(
    'your-project-slug',
    'your-api-key'
);

// Upload file from local path
$storage->uploadFromPath(
    'local-file.pdf',
    'uploads/document.pdf',
    'documents'  // bucket/folder
);

// Upload from file resource
$fileHandle = fopen('local-file.pdf', 'r');
$storage->uploadFromStream(
    $fileHandle,
    'uploads/document.pdf',
    'documents',
    'application/pdf'
);

// Check if file exists
if ($storage->fileExists('uploads/document.pdf')) {
    echo "File exists!" . PHP_EOL;
}

// Get file information
$info = $storage->getFileInfo('uploads/document.pdf');
echo "Size: " . $info['size'] . " bytes" . PHP_EOL;
echo "Modified: " . $info['last_modified'] . PHP_EOL;

// List files with prefix
$files = $storage->listFiles('uploads/2024/');
foreach ($files as $file) {
    echo $file['key'] . ': ' . $file['size'] . " bytes" . PHP_EOL;
}

// Download file (get presigned URL)
$urlData = $storage->getFileUrl('uploads/document.pdf', 3600);
echo "Download URL: " . $urlData['file_url'] . PHP_EOL;
// URL is valid for 1 hour (3600 seconds)

// Delete single file
$storage->deleteFile('uploads/old-file.pdf');

// Delete multiple files
$storage->deleteFiles([
    'uploads/file1.pdf',
    'uploads/file2.pdf',
    'uploads/file3.pdf'
]);

// Check quota
$quota = $storage->getQuota();
echo "Used: " . number_format($quota['used_gb'], 2) . " GB" . PHP_EOL;
echo "Available: " . number_format($quota['available_gb'], 2) . " GB" . PHP_EOL;
echo "Usage: " . number_format($quota['usage_percentage'], 1) . "%" . PHP_EOL;

// Check if enough storage before upload
$fileSize = filesize('large-file.zip');
if ($quota['available_bytes'] < $fileSize) {
    echo "Not enough storage!" . PHP_EOL;
} else {
    $storage->uploadFromPath('large-file.zip', 'uploads/large-file.zip');
}
```

### Error Handling

```php
try {
    $users = $client->table('users')->select('*')->get();
    echo "Success: " . count($users['data']) . " users" . PHP_EOL;
} catch (WOWSQL\AuthenticationException $e) {
    echo "Authentication error: " . $e->getMessage() . PHP_EOL;
} catch (WOWSQL\NotFoundException $e) {
    echo "Not found: " . $e->getMessage() . PHP_EOL;
} catch (WOWSQL\RateLimitException $e) {
    echo "Rate limit exceeded: " . $e->getMessage() . PHP_EOL;
} catch (WOWSQL\WOWSQLException $e) {
    echo "Database error (" . $e->getStatusCode() . "): " . $e->getMessage() . PHP_EOL;
}

try {
    $storage->uploadFromPath('file.pdf', 'uploads/file.pdf');
} catch (WOWSQL\StorageLimitExceededException $e) {
    echo "Storage full: " . $e->getMessage() . PHP_EOL;
    echo "Please upgrade your plan or delete old files" . PHP_EOL;
} catch (WOWSQL\StorageException $e) {
    echo "Storage error: " . $e->getMessage() . PHP_EOL;
}
```

### Utility Methods

```php
// List all tables
$tables = $client->listTables();
print_r($tables);

// Get table schema
$schema = $client->getTableSchema('users');
echo "Columns: " . count($schema['columns']) . PHP_EOL;
foreach ($schema['columns'] as $column) {
    echo "  - " . $column['name'] . " (" . $column['type'] . ")" . PHP_EOL;
}

// Check API health
$health = $client->health();
echo "Status: " . $health['status'] . PHP_EOL;
```

## 🔧 Configuration

### Custom Timeout

```php
// Database client
$client = new WOWSQLClient(
    'https://your-project.wowsql.com',
    'your-api-key',
    60  // 60 seconds timeout
);

// Storage client
$storage = new WOWSQLStorage(
    'your-project-slug',
    'your-api-key',
    120  // 2 minutes for large files
);
```

### Using Environment Variables

```php
// Best practice: Use environment variables for API keys
$client = new WOWSQLClient(
    getenv('WOWSQL_PROJECT_URL'),
    getenv('WOWSQL_API_KEY')
);

$storage = new WOWSQLStorage(
    getenv('WOWSQL_PROJECT_URL'),
    getenv('WOWSQL_API_KEY')
);
```

## 🔑 API Keys

WOWSQL uses **different API keys for different operations**. Understanding which key to use is crucial for proper authentication.

### Key Types Overview

| Operation Type | Recommended Key | Alternative Key | Used By |
|---------------|----------------|-----------------|---------|
| **Database Operations** (CRUD) | Service Role Key (`wowbase_service_...`) | Anonymous Key (`wowbase_anon_...`) | `WOWSQLClient` |
| **Storage Operations** | Service Role Key (`wowbase_service_...`) | Anonymous Key (`wowbase_anon_...`) | `WOWSQLStorage` |

### Where to Find Your Keys

All keys are found in: **WOWSQL Dashboard → Authentication → PROJECT KEYS**

1. **Service Role Key** (`wowbase_service_...`)
   - Location: "Service Role Key (keep secret)"
   - Used for: Database CRUD operations and storage (recommended for server-side)
   - **Important**: Click the eye icon to reveal this key

2. **Anonymous Key** (`wowbase_anon_...`)
   - Location: "Anonymous Key"
   - Used for: Public/client-side database operations with limited permissions
   - Optional: Use when exposing database access to frontend/client

### Database Operations

Use **Service Role Key** or **Anonymous Key** for database operations:

```php
// Using Service Role Key (recommended for server-side, full access)
$client = new WOWSQLClient(
    'https://your-project.wowsql.com',
    'wowbase_service_your-service-key-here'  // Service Role Key
);

// Using Anonymous Key (for public/client-side access with limited permissions)
$client = new WOWSQLClient(
    'https://your-project.wowsql.com',
    'wowbase_anon_your-anon-key-here'  // Anonymous Key
);

// Query data
$users = $client->table('users')->get();
```

### Security Best Practices

1. **Never expose Service Role Key** in client-side code or public repositories
2. **Use Anonymous Key** for public database access with limited permissions
3. **Store keys in environment variables**, never hardcode them
4. **Rotate keys regularly** if compromised

## 🎨 Framework Integration

### Laravel

```php
// config/services.php
return [
    'wowsql' => [
        'project_url' => env('WOWSQL_PROJECT_URL'),
        'api_key' => env('WOWSQL_API_KEY'),
    ],
];

// app/Services/WowSQLService.php
namespace App\Services;

use WOWSQL\WOWSQLClient;

class WowSQLService
{
    protected $client;

    public function __construct()
    {
        $this->client = new WOWSQLClient(
            config('services.wowsql.project_url'),
            config('services.wowsql.api_key')
        );
    }

    public function getClient()
    {
        return $this->client;
    }
}

// Usage in Controller
use App\Services\WowSQLService;

class UserController extends Controller
{
    public function index(WowSQLService $wowsql)
    {
        $users = $wowsql->getClient()
            ->table('users')
            ->select('id', 'name', 'email')
            ->get();
        
        return view('users.index', ['users' => $users['data']]);
    }
}
```

### Symfony

```php
// config/services.yaml
services:
    WOWSQL\WOWSQLClient:
        arguments:
            $projectUrl: '%env(WOWSQL_PROJECT_URL)%'
            $apiKey: '%env(WOWSQL_API_KEY)%'

// Usage in Controller
use WOWSQL\WOWSQLClient;

class UserController extends AbstractController
{
    public function index(WOWSQLClient $client)
    {
        $users = $client->table('users')
            ->select('id', 'name', 'email')
            ->get();
        
        return $this->json($users['data']);
    }
}
```

## 📋 Requirements

- PHP 7.4 or higher
- Composer
- Guzzle HTTP Client 7.0+

## 🔗 Links

- 📚 [Documentation](https://wowsql.com/docs)
- 🌐 [Website](https://wowsql.com)
- 💬 [Discord](https://discord.gg/WOWSQL)
- 🐛 [Issues](https://github.com/wowsql/wowsql/issues)

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## 📞 Support

- Email: support@wowsql.com
- Discord: https://discord.gg/WOWSQL
- Documentation: https://wowsql.com/docs

---

Made with ❤️ by the WOWSQL Team
