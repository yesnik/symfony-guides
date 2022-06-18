# Symfony Installation

See [docs](https://symfony.com/doc/current/setup.html)

1. Install [Symfony CLI](https://github.com/symfony-cli/symfony-cli). Follow [installation instructions](https://symfony.com/download).
2. Check if your computer meets all requirements:
    ```bash
    symfony check:requirements
    ```
3. Create Symfony app:
    ```bash
    # run this if you are building a traditional web application
    symfony new mysite --version=6.0.* --webapp

    # run this if you are building a microservice, console application or API
    symfony new mysite --version=6.0.*
    ```
