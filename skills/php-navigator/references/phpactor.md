# Semantic navigation with Phpactor (optional upgrade)

`ctags` is lexical: it indexes symbol *names* and their definition lines, but it
does not understand PHP's type system. It cannot resolve `use` aliases, follow
namespaces, or compute inheritance relationships. For most "where is X?"
navigation that is fine and fast. Reach for Phpactor only when a task genuinely
needs semantic precision, such as:

- "Which concrete classes implement this interface / extend this class?"
- Resolving a short class name to its fully-qualified name through imports.
- Distinguishing two methods with the same name on unrelated classes by their
  real owning type.

## Installing Phpactor

Phpactor is a PHP tool, so the project (or environment) needs PHP available.
Install the standalone PHAR — this does not touch the project's Composer
dependencies:

```bash
curl -Lo phpactor.phar https://github.com/phpactor/phpactor/releases/latest/download/phpactor.phar
chmod +x phpactor.phar
./phpactor.phar --version
```

## Building and querying the index

From the project root, build the index (this can take a moment on a large repo),
then query it:

```bash
./phpactor.phar index:build

# Query the index. The syntax is <kind>#<name>:
./phpactor.phar index:query "class#PaymentProcessor"
./phpactor.phar index:query "method#refund"
./phpactor.phar index:query "function#formatMoney"
```

The index updates incrementally as files change when a file watcher is
configured (Watchman is the recommended watcher); otherwise rebuild with
`index:build` after significant edits.

## Composer classmap as a lightweight middle ground

If the project uses Composer with PSR-4 autoloading, the autoloader already
knows class → file. Generating an authoritative classmap is a cheap way to
resolve a fully-qualified class name to its file without a full semantic index:

```bash
composer dump-autoload --optimize --classmap-authoritative
# Then read vendor/composer/autoload_classmap.php — it maps FQCN => file path.
```

This resolves *classes* accurately (including through PSR-4) but does not cover
methods, functions, or usages — combine it with `phpnav refs` for those.

## Choosing between the tools

- Default to `phpnav` (ctags + ripgrep): zero project dependencies, instant,
  covers the large majority of navigation.
- Add the Composer classmap when you specifically need accurate class→file
  resolution and the project is PSR-4.
- Bring in Phpactor when you need true semantic answers (implementers,
  inheritance, alias resolution) and PHP is available to run it.
