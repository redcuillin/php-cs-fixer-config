# php-cs-fixer-config

[![Continuous Deployment](https://github.com/ergebnis/php-cs-fixer-config/workflows/Continuous%20Deployment/badge.svg)](https://github.com/ergebnis/php-cs-fixer-config/actions)
[![Continuous Integration](https://github.com/ergebnis/php-cs-fixer-config/workflows/Continuous%20Integration/badge.svg)](https://github.com/ergebnis/php-cs-fixer-config/actions)

[![Code Coverage](https://codecov.io/gh/ergebnis/php-cs-fixer-config/branch/master/graph/badge.svg)](https://codecov.io/gh/ergebnis/php-cs-fixer-config)
[![Type Coverage](https://shepherd.dev/github/ergebnis/php-cs-fixer-config/coverage.svg)](https://shepherd.dev/github/ergebnis/php-cs-fixer-config)

[![Latest Stable Version](https://poser.pugx.org/ergebnis/php-cs-fixer-config/v/stable)](https://packagist.org/packages/ergebnis/php-cs-fixer-config)
[![Total Downloads](https://poser.pugx.org/ergebnis/php-cs-fixer-config/downloads)](https://packagist.org/packages/ergebnis/php-cs-fixer-config)

Provides a configuration factory and multiple rule sets for [`friendsofphp/php-cs-fixer`](http://github.com/FriendsOfPHP/PHP-CS-Fixer).

## Installation

Run

```sh
$ composer require --dev ergebnis/php-cs-fixer-config
```

## Usage

### Configuration

Pick one of the rule sets:

* [`Ergebnis\PhpCsFixer\RuleSet\Laravel6`](src/RuleSet/Laravel6.php)
* [`Ergebnis\PhpCsFixer\RuleSet\Php71`](src/RuleSet/Php71.php)
* [`Ergebnis\PhpCsFixer\RuleSet\Php73`](src/RuleSet/Php73.php)

Create a configuration file `.php_cs` in the root of your project:

```php
<?php

use Ergebnis\PhpCsFixer\Config;

$config = Config\Factory::fromRuleSet(new Config\RuleSet\Php71());

$config->getFinder()->in(__DIR__);
$config->setCacheFile(__DIR__ . '/.build/php-cs-fixer/php_cs.cache');

return $config;
```

### Git

All configuration examples use the caching feature, and if you want to use it as well, you should add the cache directory to `.gitignore`:

```diff
+ /.build/
 /vendor/
```

:bulb: Personally, I prefer to use a `.build` directory for storing build artifacts.

### Configuration with header

:bulb: Optionally specify a header:

```diff
 <?php

 use Ergebnis\PhpCsFixer\Config;

+$header = <<<EOF
+Copyright (c) 2019 Andreas Möller
+
+For the full copyright and license information, please view
+the LICENSE file that was distributed with this source code.
+
+@see https://github.com/ergebnis/php-cs-fixer-config
+EOF;

-$config = Config\Factory::fromRuleSet(new Config\RuleSet\Php71());
+$config = Config\Factory::fromRuleSet(new Config\RuleSet\Php71($header));

 $config->getFinder()->in(__DIR__);
 $config->setCacheFile(__DIR__ . '/.build/php-cs-fixer/php_cs.cache');

 return $config;
```

This will enable and configure the [`HeaderCommentFixer`](https://github.com/FriendsOfPHP/PHP-CS-Fixer/blob/v2.1.1/src/Fixer/Comment/HeaderCommentFixer.php), so that
file headers will be added to PHP files, for example:

```php
<?php

/**
 * Copyright (c) 2019 Andreas Möller
 *
 * For the full copyright and license information, please view
 * the LICENSE file that was distributed with this source code.
 *
 * @see https://github.com/ergebnis/php-cs-fixer-config
 */
```

### Configuration with override rules

:bulb: Optionally override rules from a rule set by passing in an array of rules to be merged in:

```diff
 <?php

 use Ergebnis\PhpCsFixer\Config;

-$config = Config\Factory::fromRuleSet(new Config\RuleSet\Php71());
+$config = Config\Factory::fromRuleSet(new Config\RuleSet\Php71(), [
+    'mb_str_functions' => false,
+    'strict_comparison' => false,
+]);

 $config->getFinder()->in(__DIR__);
 $config->setCacheFile(__DIR__ . '/.build/php-cs-fixer/php_cs.cache');

 return $config;
```

### Makefile

If you like [`Makefile`](https://www.gnu.org/software/make/manual/make.html#Introduction)s, create a `Makefile` with a `coding-standards` target:

```diff
+.PHONY: coding-standards
+coding-standards: vendor
+	 mkdir -p .build/php-cs-fixer
+    vendor/bin/php-cs-fixer fix --config=.php_cs --diff --verbose

 vendor: composer.json composer.lock
     composer validate
     composer install
```

Run

```
$ make coding-standards
```

to automatically fix coding standard violations.

### Composer script

If you like [`composer` scripts](https://getcomposer.org/doc/articles/scripts.md), add a `coding-standards` script to `composer.json`:

```diff
 {
   "name": "foo/bar",
   "require": {
     "php": "^7.2",
   },
   "require-dev": {
     "ergebnis/php-cs-fixer-config": "~1.0.0"
+  },
+  "scripts": {
+    "coding-standards": [
+      "mkdir -p .build/php-cs-fixer",
+      "php-cs-fixer fix --diff --diff-format=udiff --verbose"
+    ]
   }
 }
```

Run

```
$ composer coding-standards
```

to automatically fix coding standard violations.

### GitHub Actions

If you like [GitHub Actions](https://github.com/features/actions), add a `coding-standards` job to your workflow:

```diff
 on:
   pull_request:
   push:
     branches:
       - master
     tags:
       - "**"

 name: "Continuous Integration"

 jobs:
+  coding-standards:
+    name: "Coding Standards"
+
+    runs-on: ubuntu-latest
+
+    steps:
+      - name: "Checkout"
+        uses: actions/checkout@v1.1.0
+
+      - name: "Disable Xdebug"
+        run: php7.2 --ini | grep xdebug | sed 's/,$//' | xargs sudo rm
+
+      - name: "Cache dependencies installed with composer"
+        uses: actions/cache@v1.0.2
+        with:
+          path: ~/.composer/cache
+          key: php7.2-composer-locked-${{ hashFiles('**/composer.lock') }}
+          restore-keys: |
+            php7.2-composer-locked-
+
+      - name: "Install locked dependencies with composer"
+        run: php7.2 $(which composer) install --no-interaction --no-progress --no-suggest
+
+      - name: "Create cache directory for friendsofphp/php-cs-fixer"
+        run: mkdir -p .build/php-cs-fixer
+
+      - name: "Cache cache directory for friendsofphp/php-cs-fixer"
+        uses: actions/cache@v1.0.2
+        with:
+          path: ~/.build/php-cs-fixer
+          key: php7.2-php-cs-fixer-${{ hashFiles('**/composer.lock') }}
+          restore-keys: |
+            php7.2-php-cs-fixer-
+
+      - name: "Run friendsofphp/php-cs-fixer"
+        run: php7.2 vendor/bin/php-cs-fixer fix --config=.php_cs --diff --diff-format=udiff --dry-run --verbose
```

### Travis

If you like [Travis CI](https://travis-ci.com), add a `coding-standards` stage to your jobs:

```diff
 language: php

 cache:
   directories:
     - $HOME/.composer/cache
+    - .build/php-cs-fixer

 jobs:
   include:
+    - stage: "Coding Standards"
+
+      php: 7.2
+
+      install:
+        - composer install --no-interaction --no-progress --no-suggest
+
+      before_script:
+        - mkdir -p .build/php-cs-fixer
+
+      script:
+        - vendor/bin/php-cs-fixer fix --config=.php_cs --diff --dry-run --verbose
```

## Changelog

Please have a look at [`CHANGELOG.md`](CHANGELOG.md).

## Contributing

Please have a look at [`CONTRIBUTING.md`](.github/CONTRIBUTING.md).

:bulb: Do you want to add a rule for personal use or use in your organization? Instead of opening a pull request here, perhaps consider creating a new package based on [`ergebnis/php-cs-fixer-config-template`](https://github.com/ergebnis/php-cs-fixer-config-template), a GitHub repository template that provides a good starting point for creating and sharing your own rule sets.

## Code of Conduct

Please have a look at [`CODE_OF_CONDUCT.md`](https://github.com/ergebnis/.github/blob/master/CODE_OF_CONDUCT.md).

## License

This package is licensed using the [MIT License](LICENSE.md).

## Credits

This project is inspired by and also replaces [`localheinz/php-cs-fixer-config`](https://github.com/localheinz/php-cs-fixer-config).

The rule set [`Ergebnis\PhpCsFixer\RuleSet\Laravel6`](src/RuleSet/Laravel6.php) is based on [`laravel-shift/.php_cs.laravel.php`](https://gist.github.com/laravel-shift/cab527923ed2a109dda047b97d53c200).
