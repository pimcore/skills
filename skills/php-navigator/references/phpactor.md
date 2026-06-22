# Phpactor: the semantic layer

This plugin ships an LSP configuration (`.lsp.json`) that wires Claude Code to
**Phpactor**, an open-source PHP language server. Phpactor parses your code and
resolves types, namespaces, `use` aliases, and inheritance, so it answers the
questions ctags cannot: go-to-definition through imports, find-implementations
of an interface, accurate find-references, hover/type information, and
diagnostics after edits.

## Installing Phpactor

Phpactor is a PHP tool, so a PHP runtime must be available. Install the
standalone PHAR and put it on `PATH` as `phpactor` (the name the `.lsp.json`
expects):

```bash
curl -Lo phpactor.phar https://github.com/phpactor/phpactor/releases/latest/download/phpactor.phar
chmod +x phpactor.phar
sudo mv phpactor.phar /usr/local/bin/phpactor      # or anywhere on PATH
phpactor language-server --help                    # confirm it runs
```

`stdio` is Phpactor's default language-server transport, which matches the
`.lsp.json` here. For best resolution, run `composer install` in the project
first so Phpactor can index `vendor/`. After installing (or changing
`.lsp.json`), run `/reload-plugins` or restart Claude Code so the server config
is picked up; on a project-scoped plugin the language server starts only after
you accept the workspace trust dialog.

## What the `.lsp.json` sets

```json
{
  "php": {
    "command": "phpactor",
    "args": ["language-server"],
    "extensionToLanguage": { ".php": "php", ".phtml": "php" },
    "transport": "stdio",
    "startupTimeout": 30000,
    "maxRestarts": 3
  }
}
```

- `startupTimeout` is generous because Phpactor's first index of a large repo
  can take a while; the LSP handshake itself is quick.
- Diagnostics are pushed into Claude's context by default after edits. To keep
  navigation but silence automatic diagnostics, add `"diagnostics": false`.
- Phpactor can integrate PHPStan/Psalm and php-cs-fixer as diagnostic providers;
  see the Phpactor docs if you want richer diagnostics than the built-ins.

## Optional: the CLI indexer (no server running)

Phpactor also has a standalone indexer you can query directly, independent of
the LSP — handy in scripts or one-off checks:

```bash
phpactor index:build
phpactor index:query "class#PaymentProcessor"
phpactor index:query "method#refund"
```

## Composer classmap as a lightweight class resolver

For PSR-4 projects, the Composer autoloader already maps class → file. This is a
cheap way to resolve a fully-qualified class name to its file without a running
server:

```bash
composer dump-autoload --optimize --classmap-authoritative
# Read vendor/composer/autoload_classmap.php — it maps FQCN => file path.
```

It resolves classes accurately but not methods, functions, or usages — combine
it with `phpnav refs` for those.

## Choosing between the layers

- Prefer the **Phpactor LSP** for any semantic question: implementers,
  inheritance, alias resolution, accurate references, diagnostics.
- Fall back to **`phpnav`** (ctags + ripgrep) when the server isn't available,
  for quick lexical lookups, or for arbitrary string searches.
- Use the **Composer classmap** when you specifically need accurate class→file
  resolution in a PSR-4 project and the server isn't running.
