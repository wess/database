# Utopia Database

[![Build Status](https://travis-ci.org/utopia-php/abuse.svg?branch=master)](https://travis-ci.com/utopia-php/database)
![Total Downloads](https://img.shields.io/packagist/dt/utopia-php/database.svg)
[![Discord](https://img.shields.io/discord/564160730845151244)](https://appwrite.io/discord)

Utopia framework database library is simple and lite library for managing application persistency using multiple database adapters. This library is aiming to be as simple and easy to learn and use. This library is maintained by the [Appwrite team](https://appwrite.io).

Although this library is part of the [Utopia Framework](https://github.com/utopia-php/framework) project it is dependency free, and can be used as standalone with any other PHP project or framework.

## Getting Started

Install using composer:
```bash
composer require utopia-php/database
```

Initialization:

```php
<?php

require_once __DIR__ . '/../../vendor/autoload.php';

```

### Concepts

A list of the utopia/php concepts and their relevant equivalent using the different adapters

- **Database** - An instance of the utopia/database library that abstracts one of the supported adapters and provides a unified API for CRUD operation and queries on a specific schema or isolated scope inside the underlining database.
- **Adapter** - An implementation of an underlying database engine that this library can support - below is a list of [supported adapters](#supported-adapters) and supported capabilities for each Adapter.
- **Collection** - A set of documents stored on the same adapter scope. For SQL-based adapters, this will be equivalent to a table. For a No-SQL adapter, this will equivalent to a native collection.
- **Document** - A simple JSON object that will be stored in one of the utopia/database collections. For SQL-based adapters, this will be equivalent to a row. For a No-SQL adapter, this will equivalent to a native document.
- **Attribute** - A simple document attribute. For SQL-based adapters, this will be equivalent to a column. For a No-SQL adapter, this will equivalent to a native document field.
- **Index** - A simple collection index used to improve the performance of your database queries.
- **Permissions** - Using permissions, you can decide which roles will grant read or write access for a specific document. The special attributes `$read` and `$write` are used to store permissions metadata for each document in the collection. A permission role can be any string you want. You can use `Authorization::setRole()` to delegate new roles to your users, once obtained a new role a user would gain read or write access to a relevant document.

### Reserved Attributes

- `$id` - the documnet unique ID, you can set your own custom ID or a random UID will be generated by the library.
- `$collection` - an attribute containing the name of the collection the document is stored in.
- `$read` - an attribute containing an array of strings. Each string represent a specific role. If your user obtains that role he will have read access for this document.
- `$write` - an attribute containing an array of strings. Each string represent a specific role. If your user obtains that role he will have write access for this document.

### Attribute Types

The database document interface only supports primitives types (`strings`, `integers`, `floats`, and `booleans`) translated to their native database types for each of the relevant database adapters. Complex types like arrays or objects will be encoded to JSON strings when stored and decoded back when fetched from their adapters.

### Examples

Some examples to help you get started.

**Creating a database:**

```php
use PDO;
use Utopia\Database\Database;
use Utopia\Database\Adapter\MariaDB;
use Utopia\Cache\Cache;
use Utopia\Cache\Adapter\None as NoCache;

$dbHost = 'mariadb';
$dbPort = '3306';
$dbUser = 'root';
$dbPass = 'password';

$pdo = new PDO("mysql:host={$dbHost};port={$dbPort};charset=utf8mb4", $dbUser, $dbPass, [
    PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4',
    PDO::ATTR_TIMEOUT => 3, // Seconds
    PDO::ATTR_PERSISTENT => true,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
]);

$cache = new Cache(new NoCache()); // or use any cache adapter you wish

$database = new Database(new MariaDB($pdo), $cache);
$database->setNamespace('mydb');

$database->create(); // Creates a new schema named `mydb`
```

**Creating a collection:**

```php
$database->createCollection('movies');

// Add attributes
$database->createAttribute('movies', 'name', Database::VAR_STRING, 128, true);
$database->createAttribute('movies', 'director', Database::VAR_STRING, 128, true);
$database->createAttribute('movies', 'year', Database::VAR_INTEGER, 0, true);
$database->createAttribute('movies', 'price', Database::VAR_FLOAT, 0, true);
$database->createAttribute('movies', 'active', Database::VAR_BOOLEAN, 0, true);
$database->createAttribute('movies', 'generes', Database::VAR_STRING, 32, true, true, true);

// Create an Index
$database->createIndex('movies', 'index1', Database::INDEX_KEY, ['year'], [128], [Database::ORDER_ASC]);
```

**Create a document:**

```php
static::getDatabase()->createDocument('movies', new Document([
    '$read' => ['*', 'user1', 'user2'],
    '$write' => ['*', 'user1x', 'user2x'],
    'name' => 'Captain Marvel',
    'director' => 'Anna Boden & Ryan Fleck',
    'year' => 2019,
    'price' => 25.99,
    'active' => true,
    'generes' => ['science fiction', 'action', 'comics'],
]));
```

**Find:**

```php
$documents = static::getDatabase()->find('movies', [
    new Query('year', Query::TYPE_EQUAL, [2019]),
]);
```

### Adapters

Below is a list of supported adapters, and thier compatibly tested versions alongside a list of supported features and relevant limits.

| Adapter | Status | Version |
|---------|---------|---|
| MariaDB | ✅ | 10.5 |
| MySQL | ✅ | 8.0 |
| Postgres | 🛠 | 13.0 |
| MongoDB | 🛠 | 3.6 |
| SQLlite | 🛠 | 3.35 |

` ✅  - supported, 🛠  - work in progress`

## TODOS

- [ ] CRUD: Updated databases list method
- [x] CRUD: Updated collection list method
- [ ] CRUD: Add format validation on top of type validation
- [ ] CRUD: Validate original document before editing `$id`
- [ ] CRUD: Test no one can overwrite exciting documents/collections without permission
- [x] FIND: Implement or conditions for the find method
- [x] FIND: Implement or conditions with floats
- [ ] FIND: Test for find timeout limits
- [ ] FIND: Add a query validator (Limit queries to indexed attaributes only?)
- [ ] FIND: Add support for more operators (search/match/like)
- [ ] TEST: Find Limit & Offset values 
- [ ] TEST: Find Order by (+multiple attributes) 
- [ ] TEST: Missing Collection, DocumentId validators tests

## Open Issues

- Lazy index creation, maybe add a queue attribute to populate before creating the index?
- In queries for arrays, should we create a dedicated index?

## System Requirements

Utopia Framework requires PHP 7.3 or later. We recommend using the latest PHP version whenever possible.

## Tests

To run all unit tests, use the following Docker command:

```bash
docker-compose exec tests vendor/bin/phpunit --configuration phpunit.xml tests
```

To run static code analysis, use the following Psalm command:

```bash
docker-compose exec tests vendor/bin/psalm --show-info=true
```
## Authors

**Eldad Fux**

+ [https://twitter.com/eldadfux](https://twitter.com/eldadfux)
+ [https://github.com/eldadfux](https://github.com/eldadfux)

**Brandon Leckemby**

+ [https://github.com/kodumbeats](https://github.com/kodumbeats)
+ [blog.leckemby.me](blog.leckemby.me)

## Copyright and license

The MIT License (MIT) [http://www.opensource.org/licenses/mit-license.php](http://www.opensource.org/licenses/mit-license.php)
