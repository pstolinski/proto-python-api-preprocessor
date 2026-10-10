# PROTO Python API Preprocessor

This repository builds the compact Python API vocabulary used by the PROTO
website for Pyrefly linting and CodeMirror autocomplete. It analyzes Python
source statically with Griffe; robot code and CircuitPython dependencies are
never imported or executed.

The repository contains the build tools, debugger, and an example
configuration. Python library source, machine-local configuration, generated
JSON, and installed dependencies are intentionally excluded from Git.

## Requirements

Install these tools before setting up the project:

- [Node.js](https://nodejs.org/) with npm.
- [uv](https://docs.astral.sh/uv/getting-started/installation/).

The npm scripts use uv to download and cache an isolated Griffe 2.1.0
environment. A global Python or Griffe installation is not required.

## Set up a new checkout

From the repository directory:

```sh
npm install
npm run init
```

`npm run init`:

1. Copies `preprocessor.config.example.json` to the ignored
   `preprocessor.config.json`.
2. Creates the configured source and output directories when necessary.
3. Leaves an existing configuration untouched.

Next, place the Python library under the configured `sourceRoot`. With the
default configuration, the relevant layout starts like this:

```text
lib/
├── make.py
└── proto/
    ├── __init__.py
    └── ...
```

The public `make` module imports the functions and classes that should be
available to learners. Internal packages imported by those public objects must
also be present under `sourceRoot` so Griffe can resolve them.

Finally, review `preprocessor.config.json` before generating anything.

## Configuration

The live `preprocessor.config.json` is machine-local and ignored by Git.
`preprocessor.config.example.json` is the tracked starting point for new
checkouts.

| Field            | Purpose                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------- |
| `sourceRoot`     | Directory containing the Python modules Griffe analyzes.                                          |
| `output`         | Local JSON file written by `npm run generate:json`.                                               |
| `copyOutputTo`   | Destination used only by `npm run copy:json`. This may point into a neighboring website checkout. |
| `module`         | Public Python module represented by the vocabulary, normally `make`.                              |
| `pythonVersion`  | Python version supplied to Pyrefly in the website.                                                |
| `griffePackages` | Root packages Griffe must load to resolve the public module and imported implementations.         |
| `categories`     | Ordered public API allowlist. Keys are dotted public paths and values are display categories.     |

Only paths listed in `categories` enter the generated public vocabulary. Their
order is preserved. This prevents runtime-only imports and internal helpers
from appearing in learner autocomplete.

When shared defaults change, update `preprocessor.config.example.json`.
Running `npm run init` again will not overwrite anyone's live configuration.

## Generate vocabulary

```sh
npm run generate:json
```

Generation performs the following steps:

1. Validate the local configuration and source directory.
2. Run the pinned Griffe CLI with static inspection only.
3. Resolve the configured public imports into their implementation modules.
4. Extract signatures, annotations, defaults, docstrings, return types, and
   public class methods.
5. Remove private, variadic, and injected implementation parameters.
6. Write readable JSON to the configured `output`.

The adapter stops with an error if a configured public path cannot be resolved.
It ignores `self`, `cls`, variadic parameters, and the injected `schema`
constructor parameter.

The generated JSON can be reviewed or temporarily edited before copying.
Regeneration overwrites those edits. For durable changes:

- Edit docstrings and type annotations in the Python source.
- Add, remove, categorize, or reorder API entries in `categories`.
- Add a root to `griffePackages` when public implementations move into another
  internal package.

## Copy reviewed output

```sh
npm run copy:json
```

This copies the configured `output` to `copyOutputTo`. It is the only command
that writes into the consumer project. Generation and debugging remain local
to this repository.

## Debugger

To inspect whatever setup currently exists:

```sh
npm run debug
```

This starts the read-only debugger at `http://localhost:4173` without running
generation first. It can therefore start on an incomplete checkout. The page
reports whether configuration is missing or invalid, or whether vocabulary
still needs to be generated.

To regenerate first and then start the debugger:

```sh
npm run debug:fresh
```

`debug:fresh` requires a valid configuration and populated source directory.

The debugger reads `preprocessor.config.json` and the configured output
directly. It does not create another combined JSON artifact. Its server exposes
only:

- Files inside `debug/`.
- The local configuration.
- The configured generated vocabulary.
- Ellipsi's browser module from `node_modules`.

## Git-managed and local files

Commit the build scripts, debugger, README, package files, and
`preprocessor.config.example.json`.

These paths are local working state and are ignored:

```text
preprocessor.config.json
lib/
generated/
node_modules/
```

This keeps Python library checkouts, generated output, local destination paths,
and installed packages out of the preprocessor repository.
# proto-python-api-preprocessor
