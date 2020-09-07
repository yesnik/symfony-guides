# Fixtures

Do you want to populate your development database automatically?

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

*Note:* Add `-vvv` to make command's output verbose.
