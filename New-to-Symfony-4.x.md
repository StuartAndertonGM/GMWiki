Getting Married is developed with Symfony 4.x and this Wiki is going to document the things commonly interesting to developers about it

# PHPStorm

Install the PHPStorm plugin according to the IDE version
https://github.com/Haehnchen/idea-php-symfony2-plugin/blob/master/CHANGELOG.md#version-names

Down the distributable `.jar` file from [PHPStorm Plugin Page](https://plugins.jetbrains.com/plugin/7219-symfony-plugin)

This plugin offers integration like type hints on DI container like below. Hard-coded `$doctrineRegistry` type hint is no longer necessary and the IDE will resolve and interpret it automatically

```diff
- /** @var \Symfony\Bridge\Doctrine\RegistryInterface $doctrineRegistry */
$doctrineRegistry = $kernel->getContainer()->get('doctrine');
```

# Best Practices

Symfony 4.x has a shift of best practice recommendations comparing to its predecessors (e.g., 3.x).

- A [blog](http://fabien.potencier.org/) of the Symfony author - Fabien Potencier, has documented the philosophies behind Symfony
- Official Symfony [best practices doc](http://symfony.com/doc/current/best_practices/creating-the-project.html#application-bundles)

Here are some highlights:

- [Do Bundle-less application](http://fabien.potencier.org/symfony4-monolith-vs-micro.html#bundle-less-applications)

    Don't create bundle (i.e., equivalent to Magento module) for application logic. ~The only reason to create bundle is for reusability~

    **HERE IS A CATCH, WE DO CREATE BUNDLES FOR SOME REASONS**

    https://github.com/AmpersandHQ/gettingmarried/tree/development/bundles

- [Use environment variables](http://fabien.potencier.org/symfony4-best-practices.html#environment-variables) or `.env` for setting up infrastructure specific configs (e.g., mysql credentials)

    Avoid `config.yaml`

    Deprecate `parameters.yaml` and use `.env` or environment variables instead

- [Use ORM annotation](https://symfony.com/doc/current/best_practices/business-logic.html#doctrine-mapping-information)

    Avoid `doctrine.yaml`, `.xml`, or `.php` which break up ORM metadata from domain Entity

- More...

Don't be afraid to challenge the existing best practice. It changes and evolves. If we can justify the use case of a amendment, we should go for it and more importantly, document **the reasons** here so we know why we made this decision and we can adapt easily in the future when situation change.

# Generate code (e.g., entity, controller, and more)

Thanks to Symfony 4, many skeleton boiler pate code can be generated.

## Overview code generation

For example, to generate an `WeddingEvent` entity:

```bash
$ php bin/console make:entity WeddingEvent
```

Once an entity is updated / created, we need to reflect the changes in the database schema. Generate and execute migrate:

```bash
# Generate migration script
$ php bin/console make:migration

           
  Success! 
           

 Next: Review the new migration "src/Migrations/Version20180806133408.php"
 Then: Run the migration with php bin/console doctrine:migrations:migrate
 See https://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html

# Execute migration scropt
$ php bin/console doctrine:migrations:migrate
```

Alternative, for a quick and dirty local testing, force the database schema to update:

```bash
# Force database update regardless of migration script
$ php bin/console doctrine:schema --force
```

Note: this is for local testing only. You should always generate migration script and commit to the PR.

## Generate migration script

Migration script is purely a script to run during migration. It supports DI and that means you can load entity
or other dependencies in in the process and actually run against them.

Entity migration script generation is done by `doctrine/doctrine-migrations-bundle` bundle, it compares
the differences between:
- current database schema; and
- expected database schema (which resolves from entity mapping metadata)

It is developer friendly however error prone because of the inconsistent state of our local db instance during
development.

When we implement an entity, we usually will:
- make the entity;
- force the database schema to update;
- test the entity with integration test; and 
- update the entity, and repeat

This process will cause our database going out of sync however is the most suitable development process so far.

To overcome the issue and generate reliable database schema, we should reset our db before generation:

```bash
# Stop and remove local db container
$ docker stop gettingmarried_db_1; docker rm gettingmarried_db_1

# Create db container from db image
$ docker-compose --file=./docker-compose.yml up -d
```

Or, if you don't mind taking a little longer, simply:

```
$ make osx-docker-services
```

Once the db is back online (which usually takes only seconds), run:

```
# Migrate with existing migration scripts
$ php bin/console doctrine:migrations:migrate --quiet

# Generate migration script
$ php bin/console make:migration

           
  Success! 
           

 Next: Review the new migration "src/Migrations/Version20180806133408.php"
 Then: Run the migration with php bin/console doctrine:migrations:migrate
 See https://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html
```

# Upgrade application

After pulled latest changes from remote repo, run migrate command to install database changes:

```bash
# Migrate (aka upgrade)
$ php bin/console doctrine:migrations:migrate --quiet
```

# Upgrade Symfony Core

Symfony offers a clear backward compatibility matrix and promise [backward compatibility](http://symfony.com/doc/current/contributing/code/bc.html) for **minor release**.

Upgrading from `4.1.x` to `4.x` is effortless.

```bash
$ composer update "symfony/*" --with-all-dependencies
```

See: https://symfony.com/doc/current/setup/upgrade_minor.html

Symfony 4 is supported until `11/2023`. Until its end of life, we are not expecting any major effort on upgrading it.

https://github.com/AmpersandHQ/gettingmarried/issues/1

# Install from scratch

**Important: we don't usually need to start from scratch as we stick with persisted docker image. This information however is useful for setting up production environment.** 

```bash
# (TBC) Set environment variables
# - need to consult system admin when production environment is ready and find the best way to set them up
# - for the environment variables list, please see ".env.dist"
# $ source .env

# Create database
$ php bin/console doctrine:database:create
Created database `gettingmarried` for connection named default

# Migrate (aka upgrade)
$ php bin/console doctrine:migrations:migrate --quiet
```