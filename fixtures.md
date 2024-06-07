# Fixtures

[DoctrineFixturesBundle](https://symfony.com/bundles/DoctrineFixturesBundle/current/index.html) helps us to populate our development database automatically.

```bash
composer require orm-fixtures --dev
```

Add fixture file `/src/DataFixtures/ArticleFixtures.php`:

```php
namespace App\DataFixtures;
use App\Entity\Article;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\Persistence\ObjectManager;

class ArticleFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        $article = new Article();
        $article->setTitle('Why Asteroids Taste Like Bacon');
        $manager->persist($article);
        $manager->flush();
    }
}
```

**Generate fixture**

This command allows us to create fixture:

```bash
symfony console make:fixture

#> CommentFixture
```

This command will create file `src/DataFixtures/CommentFixture.php`.

**Load fixtures**

Run command to purge database and load fixtures to database:

```bash
symfony console doctrine:fixtures:load
```
Load fixtures in the test database:

```bash
php bin/console doctrine:fixtures:load --env=test
```
Ensure that you defined connection to you test DB in `.env.test` (in this case we have DB in the Docker):

```
DATABASE_URL=postgres://main:main@127.0.0.1:32769/crm_test?sslmode=disable&charset=utf8
```

*Note:* Add `-vvv` to make command's output verbose.

### Foundry

We can generate fixtures easier with [zenstruck / foundry](https://github.com/zenstruck/foundry).
It allows to create factories for Entities and define default values.

```php
class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        QuestionFactory::new()->createMany(30);
        $manager->flush();
    }
}
```
