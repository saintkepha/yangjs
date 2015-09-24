# yfc -- YangForge Controller

`yfc` is the command shell for the YangForge framework, providing
schema-driven application lifecycle management.

`YangForge` provides runtime JavaScript execution based on YANG schema
modeling language as defined in IETF drafts and standards
([RFC 6020](http://tools.ietf.org/html/rfc6020)).

Basically, the framework enables YANG schema language to *become* a
**programming** language.

It also utilizes YAML with custom tags to construct a portable module
with embedded code.

It is written primarily using [CoffeeScript](http://coffeescript.org)
and runs on [Node.js](http://nodejs.org).

This software is **sponsored** by
[ClearPath Networks](http://www.clearpathnet.com) on behalf of the
[OPNFV](http://opnfv.org) (Open Platform for Network Functions
Virtualization) community.

Please note that this project is under **active development**. Be sure
to check back often as new updates are being pushed regularly.

  [![NPM Version][npm-image]][npm-url]
  [![NPM Downloads][downloads-image]][downloads-url]

## Welcome to 0.10.x

`YangForge` has undergone extensive restructuring with the
introduction of **YAML** as the primary application packaging
construct. This means that *aside* from the `YangForge` package itself
having NPM dependency, other generated modules no longer require NPM
for `build/publish` activities. The results are rather dramatic, as
the entire source tree has been collapsed (one flat directory) since
there is no longer any need to maintain separate directories to
individually package specific YANG module with code logic.

Furthermore, interfaces for programmatic usage of the `YangForge`
module as a library has been greatly enhanced, including asynchronous
`import` capability, which can now also load YAML/YANG/JSON files from
remote web sources. Please refer to below updated documentation for
more details.

## Installation
```bash
$ npm install -g yangforge
```

You must have `node >= 0.10.28` as a minimum requirement to run
`yangforge`.

## Usage
```
  Usage: yfc [options] [command]


  Commands:

    build [options] [file]      package the application for deployment (planned)
    config                      manage yangforge service configuration (planned)
    deploy                      deploy application into yangforge endpoint (planned)
    info [options] [name]       shows info about a specific module
    publish [options]           publish package to upstream registry (planned)
    run [options] [modules...]  runs one or more modules and/or schemas
    schema [options] [file]     process a specific YANG schema file
    sign                        sign package to ensure authenticity (planned)

  YANG driven JS application builder

  Options:

    -h, --help     output usage information
    -V, --version  output the version number
    --no-color     disable color output
```

The `yfc` command-line interface is **runtime-generated** according to
[yangforge.yang](yangforge.yang) schema definitions.  Please refer to
the schema section covering various `rpc` extension statements and
sub-statement definitions for a reference regarding different types of
command-line arguments, descriptions, and options processing syntax.
The corresponding **actions** for each of the `rpc` extensions are
implemented inside the `YangForge` YAML module
[yangforge.yaml](yangforge.yaml).

## Bundled YANG modules

There are a number of YANG schema modules commonly referenced and
utilized by other YANG modules during schema definition and they have
been *bundled* together into the `yangforge` package for convenience.
All you need to do is to `import <module name>` from your YANG schema
and they will be retrieved/resolved automatically.

name | description | reference
--- | --- | ---
[complex-types](yang/complex-types) | extensions to model complex types and typed instance identifiers | RFC-6095
[iana-crypt-hash](yang/iana-crypt-hash) | typedef for storing passwords using a hash function | RFC-7317
[ietf-inet-types](yang/ietf-inet-types) | collection of generally useful types for Internet addresses | RFC-6991
[ietf-yang-types](yang/ietf-yang-types) | collection of generally useful derived data types | RFC-6991

Additional YANG modules will be bundled into the `yangforge` package
over time. Since `yangforge` facilitate *forging* of new YANG modules
and easily using them in your own projects, only industry standards
based YANG schema modules will be considered for native bundling at
this time.

## Bundled YANG features

name | description | dependency
--- | --- | ---
[cli](features/cli.yaml) | generates command-line interface | none
[express](features/express.yaml) | generates HTTP/HTTPS web server instance | none
[restjson](features/restjson.yaml) | generates REST/JSON web services interface | express
[autodoc](features/autodoc.yaml) | generates self-documentation interface | express

You can click on the *name* entry above for reference documentation on
each feature module.

## Common Usage Examples

### Using the `schema` command
```
$ yfc schema -h


  Usage: schema [options] [file]

  process a specific YANG schema file

  Options:

    -h, --help             output usage information
    -e, --eval [string]    pass a string from the command line as input
    -f, --format [string]  specify output format (yaml, json) (default: yaml)
    -o, --output [string]  set the output filename for compiled schema
```

You can `--eval` a YANG schema **string** directly for dynamic parsing:
```bash
$ yfc schema -e 'module hello-world { description "a test"; leaf hello { type string; default "world"; } }'
```
```yaml
module:
  hello-world:
    description: a test
    leaf:
      hello:
        type: string
        default: world
```
You can specify explicit output `--format` (default is YAML as above):
```bash
$ yfc schema -e 'module hello-world { description "a test"; leaf hello { type string; default "world"; } }' -f json
```
```json
{
 "module": {
   "hello-world": {
     "description": "a test",
     "leaf": {
       "hello": {
         "type": "string",
         "default": "world"
       }
     }
   }
 }
}
```

The `schema` command performs `preprocess` stages on the passed in
YANG schema *file* which includes any `include/import` statements
found within the schema and performing all schema manipulations such
as *grouping*, *uses*, *refine*, *augment*, etc. Basically, it will
flag any validation errors while producing an output that should
represent what the schema would look like just before `compile`.

### Using the `run` command

The real power of `YangForge` is actualized when **yangforged**
modules are run using one or more **dynamic interface
generators**.

```bash
$ yfc run -h

  Usage: run [options] [modules...]

  runs one or more modules and/or schemas

  Options:

    -h, --help          output usage information
    --cli               enables commmand-line-interface
    --express [number]  enables express web server on a specified port (default: 5000)
    --restjson          enables REST/JSON interface (default: true)
    --autodoc           enables auto-generated documentation interface (default: false)
```

#### Running a dynamically *compiled* schema instance

You can `run` a YANG schema **file** and instantiate it immediately:
```bash
$ yfc run examples/jukebox.yang
express: listening on 5000
restjson: binding forgery to /restjson
```
Once it's running, you can issue HTTP calls:
```bash
$ curl localhost:5000/restjson/example-jukebox
```
```json
{
  "jukebox": {
    "library": {
      "artist": []
    },
    "player": {},
    "playlist": []
  }
}
```

The `restjson` interface dynamically routes nested module/container hierarchy:
```bash
$ curl localhost:5000/restjson/example-jukebox/jukebox/library
```
```json
{
  "artist": []
}
```

#### Running a *forged* module

You can run a *forged* module (packaged with code behaviors) as follows:
```bash
$ yfc run examples/ping.yaml
express: listening on 5000
restjson: binding forgery to /restjson
```

The example `ping` module for this section is available
[here](examples/ping.yaml). It is based on
[ping.yang](examples/ping.yang) YANG schema.

Once it's running, you can issue HTTP REPORT call to discover
capabilities of the [ping](examples/ping.yaml) module:
```bash
$ curl -X REPORT localhost:5000/restjson/ping
```
```json
{
  "name": "ping",
  "description": "An example ping module from ODL",
  "license": "MIT",
  "keywords": [
    "yangforge",
    "ping",
    "example"
  ],
  "schema": {
    "prefix": "ping",
    "namespace": "urn:opendaylight:ping",
    "revision": {
      "2013-09-11": {
        "description": "TCP ping module"
      }
    },
    "import": {
      "ietf-inet-types": {
        "prefix": "inet"
      }
    }
  },
  "operations": {
    "send-echo": "Send TCP ECHO request"
  }
}
```
You can get usage info on an available RPC call with OPTIONS:
```bash
$ curl -X OPTIONS localhost:5000/restjson/ping/send-echo
```
The below output provides details on the expected
`input/output` schema for invoking the RPC call.
```json
{
  "POST": {
    "description": "Send TCP ECHO request",
    "input": {
      "leaf": {
        "destination": {
          "type": "inet:ipv4-address"
        }
      }
    },
    "output": {
      "leaf": {
        "echo-result": {
          "type": {
            "enumeration": {
              "enum": {
                "reachable": {
                  "value": 0,
                  "description": "Received reply"
                },
                "unreachable": {
                  "value": 1,
                  "description": "No reply during timeout"
                },
                "error": {
                  "value": 2,
                  "description": "Error happened"
                }
              }
            }
          },
          "description": "Result types"
        }
      }
    }
  }
}
```
You can then try out the available RPC call as follows:
```bash
$ curl -X POST localhost:5000/restjson/ping/send-echo -H 'Content-Type: application/json' -d '{ "destination": "8.8.8.8" }'
```
```json
{
  "echo-result": "reachable"
}
```
As you would expect, it will also perform *validations* for the input based on `inet:ipv4-address` definition:

```bash
$ curl -X POST localhost:5000/restjson/ping/send-echo -H 'Content-Type: application/json' -d '{ "destination": "abcdefg" }'
```
```json
{
  "error": "AssertionError: unable to validate passed-in 'abcdefg' as 'inet:ipv4-address'"
}
```

#### Running *arbitrary* mix of modules (even **remote** sources)

The `run` command allows you to pass in as many modules as you want to
instantiate. The following example will also *listen* on a different
port.
```bash
$ yfc run --express 5555 examples/jukebox.yang examples/ping.yaml
express: listening on 5555
restjson: binding forgery to /restjson
```
You can also dynamically retrieve/run modules from **remote** systems.
```bash
$ yfc run github:saintkepha/yangforge/master/examples/jukebox.yang
express: listening on 5000
restjson: binding forgery to /restjson
```

In the example above, **github:** is simply a short-hand for
https://raw.githubusercontent.com so you can retrieve any arbitrary
YAML/YANG modules from the web (http/https) and give things a go.

The **remote** fetching capability is internally invoking the `import`
asynchronous promise routine and you can use it with the `yfc info`
command as well.

#### Running `YangForge` natively as a stand-alone instance

When you issue `run` without any target module(s) as argument, it runs
the internal `YangForge` module using defaults:

```bash
$ yfc run
express: listening on 5000
restjson: binding forgery to /restjson
```

Once it's running, you can inquire about its capabilities by issuing
HTTP REPORT call (similar output available via CLI using `yfc info`):

```bash
$ curl -X REPORT localhost:5000/restjson/yangforge
```
```json
{
  "name": "yangforge",
  "description": "YANG driven JS application builder",
  "license": "Apache-2.0",
  "keywords": [
    "build",
    "config",
    "datastore",
    "datamodel",
    "forge",
    "model",
    "yang",
    "opnfv",
    "parse",
    "restjson",
    "restconf",
    "rpc",
    "translate",
    "yang-json",
    "yang-yaml",
    "yfc"
  ],
  "schema": {
    "prefix": "yf",
    "description": "This module provides YANG v1 language based schema compilations.",
    "revision": {
      "2015-09-23": {
        "description": "Enhanced with 0.10.x functionality"
      },
      "2015-05-04": {
        "description": "Initial revision",
        "reference": "RFC-6020"
      }
    },
    "organization": "ClearPath Networks NFV R&D Group",
    "contact": "Web:  <http://www.clearpathnet.com>\nCode: <http://github.com/clearpath-networks/yangforge>\n\nAuthor: Peter K. Lee <mailto:plee@clearpathnet.com>",
    "include": "yang-v1-extensions"
  },
  "features": {
    "cli": "When enabled, generates command-line interface for the module",
    "express": "When enabled, generates HTTP/HTTPS web server instance for the module",
    "restjson": "When enabled, generates REST/JSON web services interface for the module"
  },
  "operations": {
    "build": "package the application for deployment",
    "config": "manage yangforge service configuration",
    "deploy": "deploy application into yangforge endpoint",
    "info": "shows info about a specific module",
    "publish": "publish package to upstream registry",
    "run": "runs one or more modules and/or schemas",
    "schema": "process a specific YANG schema file",
    "sign": "sign package to ensure authenticity",
    "enable": "enables passed-in set of feature(s) for the current runtime",
    "disable": "disables passed-in set of feature(s) for the current runtime",
    "infuse": "absorb requested target module(s) into current runtime",
    "defuse": "discard requested target module(s) from current runtime",
    "export": "export existing target module for remote execution"
  }
}
```

There are now a handful of *new operations* available in the context
of the `express/restjson` interface that was previously hidden in the
`cli` interface.

The `enable/disable` operations allow runtime control of various
features to be toggled on/off. Additionally, by utilizing
`infuse/defuse` operations, you can **dynamically** load/unload
modules into the runtime context. This capability allows the
`yangforge` instance to operate as an agent which can run any
*arbitrary* schema/module instance on-demand. 

Now with the `0.10.x` remote import features, you can dynamically
`infuse` any YAML/YANG into a running `yangforge` instance without any
package download/install into filesystem nor any restart of the
running process.

The `run` command internally utilizes the `infuse` operation to
instantiate the initial running process.

## Troubleshooting

When you encounter errors or issues while utilizing the `yfc` command
line utility, you can set ENVIRONMENTAL variable `yfc_debug=1` to get
complete debug output of the `YangForge` execution log.

```bash
$ yfc_debug=1 yfc <some-command>
```

The output generated is very verbose and may or may not assist you in
determining the root cause. However, when reporting an issue into the
Github repository, it will be helpful to paste a snippet of the debug
output for quicker resolution by the project maintainer.

## Using YangForge Programmatically

```coffeescript
forge = require 'yangforge'
forge.import 'my-cool-schema.yang'
.then (app) ->
  console.log app.info()
```

### Key Features

* **Parse** YAML/YANG schema files and generate runtime JavaScript
  semantic object tree hierarchy
* **Import/Export** capabilities to load modules using customizable
  importers based on regular expressions and custom import
  routines. Ability to serialize module meta data into JSON format
  that is portable across systems. Also exports serialized JS
  functions as part of export meta data.
* **Runtime Generation** allows compiler to directly create a live JS
  class object definition so that it can be instantiated via `new`
  keyword and used immediately
* **Dynamic Extensions** enable compiler to be configured with
  alternative `resolver` functions to change the behavior of produced
  output

[YangForge](yangforge.coffee) itself is also a YANG schema
([yangforge.yang](./yangforge.yang)) **compiled** module. It is
compiled by the [yang-compiler](yang-compiler.litcoffee) and
natively includes
[yang-v1-extensions](yang-v1-extensions.yaml) submodule for
supporting the YANG version 1.0
([RFC 6020](http://tools.ietf.org/html/rfc6020)) specifications.
Please reference the
[yang-v1-extensions documentation](yang-v1-extensions.md) for
up-to-date info on YANG 1.0 language coverage status. It serves as a
good reference for writing new compilers, custom extensions, custom
typedefs, among other things.

### Primary interfaces

name | description
--- | --- | ---
forge.import     | local/remote async loading of one or more modules using filenames
forge.load       | local async/sync loading of module(s), only one-at-a-time with sync
forge.compile    | local sync compilation of a module, generates class obj that can be instantiated
forge.preprocess | local sync preprocessing of a module, constraint validations, schema manipulations
forge.parse      | local sync parsing of a module, syntax validations, custom-tag resolutions

### Programmatic Usage Examples

The below examples can be executed using CoffeeScript REPL by running
`coffee` at the command-line from the top-directory of this repo.

Using the native YangForge module as a library:

```coffeescript
forge = require 'yangforge'

yang = """
  module hello-world {
    description "a test";
    leaf hello { type string; default "world"; }
  }
  """

# asynchronous load YANG schema
forge.load schema: yang
.then (app) ->
  console.log app.get 'hello-world.hello'
  app.set 'hello-world.hello', 'goodbye'
  console.log app.get()

# synchronous load YANG schema
app = forge.load schmea: yang, async: false
console.log app.info format: 'json'

# compile YANG schema model as class object
HelloWorld = forge.compile schema: yang
hello = new HelloWorld 'hello-world': hello: 'howdy there'
console.log hello.get()

yaml = """
  name: hello
  schema: !yang/schema |
    module embedded-world { leaf wow { type number; default 0; } }
  config: !json |
    { "embedded-world": { "wow": 2 } }
  """

# it can do YAML quite well
forge.load yaml
.then (app) ->
  console.log app.get()

# sometimes you just want to preprocess to see what it becomes
console.log forge.preprocess yaml
```

Forging a new module/application for build/publish using YAML
(see also [complex-types](complex-types.yaml):
```coffeescript
name: some-new-application
schema: !yang/schema some-new-application.yang
rpc:
  something-useful: !coffee/function
    (input, output, done) ->
	  console.log input.get()
	  output.set 'important data'
	  done()
```

Forging a new interface generator using YAML (see also
[cli example](features/cli.yaml)):
```yaml
name: some-new-interface
description: Some new awesome interface
run: !coffee/function
  (model, options) ->
    # code logic to dynamically construct a new interface based on passed-in context
    console.log model
```

There are many other ways of interacting with the module's class
object as well as the instantiated class.

**More examples coming soon!**

## Literate CoffeeScript Documentation

The source code is documented in Markdown format. It's code combined
with documentation all-in-one.

* [YangForge](yangforge.coffee)
  * [Compiler](yang-compiler.litcoffee)
  * [Features](features)
  * [Module](yangforge.yaml)
  * [Schema](yangforge.yang)
* External Dependencies
  * [data-synth library](http://github.com/saintkepha/data-synth)
  * [js-yaml library](https://github.com/nodeca/js-yaml)
  * [yang-parser library](https://gitlab.labs.nic.cz/labs/yang-tools/wikis/coffee_parser)

## License
  [Apache 2.0](LICENSE)

[npm-image]: https://img.shields.io/npm/v/yangforge.svg
[npm-url]: https://npmjs.org/package/yangforge
[downloads-image]: https://img.shields.io/npm/dm/yangforge.svg
[downloads-url]: https://npmjs.org/package/yangforge
