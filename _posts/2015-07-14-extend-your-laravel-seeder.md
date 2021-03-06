---
id: 35
title: Extend your Laravel Seeder
date: 2015-07-14T22:12:08+00:00
author: Maxim Lanin
layout: post
permalink: /extend-your-laravel-seeder/
dsq_thread_id:
  - 4325527042
categories:
  - Laravel
  - PHP
tags:
  - csv
  - laravel
  - php
  - seeder
  - seeding
format: standard
---
> Laravel brings you awesome seeding functionality. But it is not as perfect as it could be. By default Laravel Seeder was made to store random data (it even suggests you to use Faker package). But what will you do if you have lots of data that already exists in my DB? There are several ways to achieve this. You can import raw arrays, SQL data, or use CSV files. I choose the third approach because CSV files are easier to edit. That's why I created a [Laravel-Extend-Seeder](https://github.com/mlanin/laravel-extend-seeder) package that implements this method.

<!-- more -->

## Installation

[PHP](https://php.net) 5.4+ or [HHVM](http://hhvm.com) 3.3+, [Composer](https://getcomposer.org) and [Laravel](http://laravel.com) 5.0+ are required.

To get the latest version of Laravel-Extend-Seeder, simply add the following line to the require block of your `composer.json` file.

    "lanin/laravel-extend-seeder": "dev-master"


You'll then need to run `composer install` or `composer update` to download it and have the autoloader updated.

Once it was installed you don't have to register any ServiceProvider, Facade or publish any configs.

All you have to do is to extend your base Seeder class with `\Lanin\ExtendSeeder\Seeder` and you are good to go!

```php
<?php
class Seeder extends \Lanin\ExtendSeeder\Seeder { }
```

## Preparing

Every time you try to seed lots of data, you have to unguard your models. Now you don't have to do it.

They will be unguarded automatically in the boot method, that fires right on the start.

Also it will disable query log, that extremely slows data import.

### Environment protection

If you are afraid to corrupt your production database with seeding it with test data, even if artisan asks you if you want to do it,

you can specify `protected $environment` variable in your seeder. Boot method will check if current environment doesn't match the set one

and throw an exception, so you can be sure you don't break your production data.

## Seeding

After you extended your base seeder, you will receive two additional methods

#### seedModel($model)

The best place for this method is the `run` method of your main DatabaseSeeder.

By default you have to fire `call` method that will resolve a specified seeder as `{$tableName}TableSeeder` and launch it, and there resolve your models and do all the stuff.

This method will do it for you. It will find related seeder by model's table and pass your model to it.

**Hint:** This method returns resolved seeder object, so you can chain your seed methods.

**Example:**

```php
<?php
class Account extends \Illuminate\Database\Eloquent\Model {
  protected $table = 'accounts';
}

class DatabaseSeeder extends Seeder {

  /**
   * Seed your database.
   */
  public function run()
  {
    $this->seedModel(\App\Account::class);
  }
}

class AccountsTableSeeder extends Seeder {

  /**
   * Seed model.
   */
  public function run()
  {
    $this->getModel()->truncate();
  }
}
```

#### seedWithCsv($csvFile = '', $model = null)

This method seeds your database with data from the related csv files.

By default, it tries to find them in your `/database/seeds/csv` directory.

Files have to be names as `{$databaseName}_{$tableName}.csv`, but you can always overwrite this behaviour.

If you are calling this method via table seeder, that was called via `seedModel`, it already knows everything about your model and can resolve related csv file iteslf.

**Example:**

```php
<?php
class AccountsTableSeeder extends Seeder {

  /**
   * Seed model with CSV.
   */
  public function run()
  {
    $this->seedWithCsv();
  }
}
```

Full example you can find in the `tests/CsvSeederTest.php`

## Hints

You can leave TableSeeders empty and chain csv import via basic seeder.

```php
<?php
class DatabaseSeeder extends Seeder {

  /**
   * Seed your database.
   */
  public function run()
  {
    $this->seedModel(\App\Account::class)->seedWithCsv();
  }
}

class AccountsTableSeeder extends Seeder {

  /**
   * Seed model.
   */
  public function run()
  {

  }
}
```

You don't even have to create TableSeeders and seed right in your basic seeder.

```php
<?php
class DatabaseSeeder extends Seeder {

  /**
   * Seed your database.
   */
  public function run()
  {
    $this->setModel(\App\Account::class)->seedWithCsv();
  }
}
```

If you want to have your csv files to be set in another location you can use static method `setCsvPath($csvPath)`, where you can specify relative from your base_path path.

## Contributing

Please feel free to fork this package and contribute by submitting a pull request to enhance the functionalities.

<a class="github-button" href="https://github.com/mlanin/laravel-extend-seeder" data-icon="octicon-star" data-count-href="/mlanin/laravel-extend-seeder/stargazers" data-count-api="/repos/mlanin/laravel-extend-seeder#stargazers_count" data-count-aria-label="# stargazers on GitHub" aria-label="Star mlanin/laravel-extend-seeder on GitHub">Star</a> <a class="github-button" href="https://github.com/mlanin/laravel-extend-seeder/fork" data-icon="octicon-repo-forked" data-count-href="/mlanin/laravel-extend-seeder/network" data-count-api="/repos/mlanin/laravel-extend-seeder#forks_count" data-count-aria-label="# forks on GitHub" aria-label="Fork mlanin/laravel-extend-seeder on GitHub">Fork</a>
