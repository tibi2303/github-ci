# github-ci

Repository for centralising GitHub Actions reusable workflows for OpenEuropa components.

## Available Workflows

### Drupal CI (`drupal-ci.yml`)

Reusable workflow for Drupal modules. Includes Docker image caching, Drupal core matrix testing, Composer patch compatibility checks, and support for PHPUnit and Behat tests.

Before the automated-test matrix starts, the workflow checks whether the root package declares both `cweagans/composer-patches` and non-empty root patches. When it does, a single preflight job runs `composer install` with Composer Patches 1.x and 2.x to verify that every patched package can be installed and patched. Root runtime requirements and patched development requirements are included; unrelated development requirements are excluded. No tests are duplicated. Projects without the plugin or without root patches skip the installation steps and continue with the regular test matrix.

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `php_versions` | string | `'["8.3"]'` | JSON array of PHP versions to test |
| `core_versions` | string | `'["~10.4.0", "~10.5.0", "~11.1.0", "~11.2.0"]'` | JSON array of Drupal core versions to test |
| `run_phpunit` | boolean | `true` | Whether to run PHPUnit tests |
| `run_phpunit_batches` | boolean | `false` | Whether to run PHPUnit tests in parallel batches via the `toolkit:test-phpunit-batches` command |
| `run_behat` | boolean | `false` | Whether to run Behat tests |

#### Usage

Basic usage with all defaults:

```yaml
name: ci
on:
  pull_request:
  push:
    branches: [master]

jobs:
  ci:
    uses: openeuropa/github-ci/.github/workflows/drupal-ci.yml@main
```

With PHPUnit running in parallel batches:

```yaml
name: ci
on:
  pull_request:
  push:
    branches: [master]

jobs:
  ci:
    uses: openeuropa/github-ci/.github/workflows/drupal-ci.yml@main
    with:
      run_phpunit_batches: true
```

Running in batches requires the component to provide the `toolkit:test-phpunit-batches` command (see for example `oe_translation`), to declare the batch names in `runner.yml.dist` under `toolkit.test.phpunit.batches` and to assign every test to a batch with a PHPUnit `@group` annotation matching a batch name. The command fails if any test is not assigned to a batch.

With Behat tests enabled:

```yaml
name: ci
on:
  pull_request:
  push:
    branches: [master]

jobs:
  ci:
    uses: openeuropa/github-ci/.github/workflows/drupal-ci.yml@main
    with:
      run_behat: true
```

With custom PHP and core versions:

```yaml
name: ci
on:
  pull_request:
  push:
    branches: [master]

jobs:
  ci:
    uses: openeuropa/github-ci/.github/workflows/drupal-ci.yml@main
    with:
      php_versions: '["8.3", "8.4"]'
      core_versions: '["~10.5.0", "~11.2.0"]'
      run_behat: true
```

---

### PHP Library CI (`php-library-ci.yml`)

Reusable workflow for standalone PHP libraries (non-Drupal). Includes Docker image caching and support for PHPUnit and Behat tests.

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `php_versions` | string | `'["8.3"]'` | JSON array of PHP versions to test |
| `run_phpunit` | boolean | `true` | Whether to run PHPUnit tests |
| `run_behat` | boolean | `false` | Whether to run Behat tests |

#### Usage

Basic usage with all defaults:

```yaml
name: ci
on:
  pull_request:
  push:
    branches: [main]

jobs:
  ci:
    uses: openeuropa/github-ci/.github/workflows/php-library-ci.yml@main
```

With Behat tests (e.g., for behat-transformation-context):

```yaml
name: ci
on:
  pull_request:
  push:
    branches: [master]

jobs:
  ci:
    uses: openeuropa/github-ci/.github/workflows/php-library-ci.yml@main
    with:
      run_phpunit: false
      run_behat: true
```

---

## Prerequisites

For components to use these workflows, their `docker-compose.yml` must use the standard image:

```yaml
services:
  web:
    image: fpfis/httpd-php:8.3-dev
    # ...
```

The workflows will automatically modify this to use the `-ci` variant with the appropriate PHP version during CI runs.
