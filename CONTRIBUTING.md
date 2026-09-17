# Contributing

Use PHP 8.4 or later and Composer. Clone the repository, run `composer install`,
then run the same checks before submitting a change:

```sh
composer qa
```

This runs PHP CS Fixer in dry-run mode, PHPStan, and PHPUnit. Use
`composer cs:fix` to apply formatting. For progression changes, cover boundary
steps, inverse lookup, and floating-point tolerances. Update the relevant
[scale guide](docs/scales/index.md) and changelog when public behavior changes.

Documentation examples should include inputs and their actual output. Verify
numeric tables by executing their examples, keeping approximations explicit.
