# Console commands

Show available commands:

```
./bin/console
```

### Create project

*Way 1*

- Install [Symfony CLI](https://symfony.com/download):
- Create new project:
    ```
    symfony new mysite
    ```

*Way 2*

```
composer create-project symfony/skeleton mysite
```

### Start development server

*Way 1:*
```bash
symfony serve
symfony server:start -d
```
This command will start [Symfony Local web server](https://symfony.com/doc/current/setup/symfony_server.html)

*Way 2:*
```bash
php -S 127.0.0.1:8000 -t public
```

### Add package

```
composer require symfony/dotenv
```

Add package to dev environment only

```
composer require maker --dev
```

### Remove package

```
composer remove symfony/dotenv
```

### Create controller

```
php bin/console make:controller
```

If you get the error *There are no commands defined in the "make" namespace.*, install package:

```
composer require maker --dev
```

File of controller `src/Controller/PartnerController.php`:

```php
class PartnerController extends AbstractController
{
    /**
     * @Route("/partners", name="partner")
     */
    public function index(EntityManagerInterface $em)
    {
        $repository = $em->getRepository(Partner::class);
        $partners = $repository->findAll();

        return $this->render('partners/index.html.twig', [
            'partners' => $partners,
        ]);
    }
}
```

Store the application templates in the `templates/` directory at the root of your project.

File of template `templates/partners/index.html.twig`:

```twig
{% extends 'base.html.twig' %}

{% block content %}
    <h1>Partners</h1>

    {% if partners %}
        <ul>
        {% for partner in partners %}
            <li>{{ partner.name }} - {{ partner.email }}</li>
        {% endfor %}
        </ul>
    {% else %}
        <p>No partners found</p>
    {% endif %}

{% endblock %}
```

### Create Entity, add column, add field

```bash
symfony console make:entity
```

### Create CRUD for entity

```
symfony console make:crud
```

This command will create controller, form, templates for entity.

### Show available routes

```bash
symfony console debug:router
```

### Show available services that we can include in our app:

```
./bin/console debug:autowiring
```

If you want to search available services that match the given word *log*:

```
./bin/console debug:container log
```

If you also want to see private services:

```
./bin/console debug:container --show-private
```

### Show parameters values

```
./bin/console debug:container --parameters
```

*Useful parameters*:

- `kernel.debug` - it's `true` most of the time, but is `false` in the prod environment.

### Show current config for the bundle

```
php bin/console debug:config framework
php bin/console debug:config api_platform
```

### Dump available config options for the bundle

```bash
php bin/console config:dump-reference security
php bin/console config:dump knp_markdown
```

### Show background workers

```bash
# List all background workers managed for the current project
symfony server:status
```

### PostgreSQL

**Run psql console**

The Symfony CLI automatically detects the Docker services running for the project and exposes the environment variables that psql needs to connect to the database.

```bash
symfony run psql
```

**Create dump**

```bash
symfony run pg_dump --data-only > dump.sql
```

**Restore db from dump**

```bash
symfony run psql < dump.sql
```

### Clear app's cache

```
./bin/console cache:pool:clear cache.app
```

Clear Symfony's internal cache that helps your app run:

```
./bin/console cache:clear
```

### Create cache files

```
./bin/console cache:warmup
```

This command will create all cache files that Symfony will ever need.

### Show environment variables

```
./bin/console about
```

### Show twig filters, functions

```
php bin/console debug:twig
```

### Generate User, UserRepository

```php
php bin/console make:user
```

Depending on your answers, the command will create a `User` class/entity and update your `security.yaml` file to configure a secure password encoder (if needed) and a user provider. 

### Create database

```bash
php bin/console doctrine:database:create
```

### Drop database

```bash
php bin/console doctrine:schema:drop --force
```

### Generate migration file

**Create migration manually**

This command analyzes Entity's changes and creates the new migration file.

```bash
php bin/console make:migration
```

This command will create file `src/Migrations/Version20200210070413.php`.

**Generate migration based on schema changes**

```bash
php bin/console doctrine:migrations:diff
```

### Apply migration

```bash
php bin/console doctrine:migrations:migrate
```

### Rollback migration

```bash
php bin/console doctrine:migrations:migrate prev
```

### Show status of migrations

```bash
php bin/console doctrine:migrations:status
```

### Skip migration

You can skip single migrations by explicitly adding them to the `migration_versions` table:

```bash
php bin/console doctrine:migrations:version YYYYMMDDHHMMSS --add
```

### Execute SQL query from console

This command allows us to execute query from console:

```bash
php bin/console doctrine:query:sql 'SELECT * FROM comment'
```

## Create console command

Install bundle:

```
composer require maker --dev
```

Create console command, enter name in dialog `article:stats`:

```
php bin/console make:command
```

It will create a file `src/Command/ArticleStatsCommand.php`.

We can run our new command:

```
bin/console article:stats
```
