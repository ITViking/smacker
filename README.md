smacker
=======

Don't smack your developers when a process misbehaves. It's hard to know everything a Node.js process does.

## Did you know?
- Packages you use can emit warnings on [`process.emitWarning`](https://nodejs.org/api/process.html#process_process_emitwarning_warning_type_code_ctor) for deprecation notices and unexpected usage, without affecting functionality.
- Both [`SIGINT and SIGTERM`] will default to try to kill your process, without notifying you. This can be confusing when running your Node.js process in foreign environments.
- Node.js v10.12.0 introduced a new event called [`multipleResolves`](https://nodejs.org/api/process.html#process_event_multipleresolves) - which they recommend should terminate your process, even though it doesn't default to that behaviour.

**smacker gracefully handles all these nitty gritties, and fits perfectly into a microservice environment.**


## Usage

``` javascript
const smacker = require('smacker');
const Service = require('lib/Service');
const service = new Service();
smacker.start({ service });
```

Check out the demo in `test/demo.js`.

### smacker.start({ service[, config][, log] })

- `service` `<Object>` must have a `start` and `stop` function, both returning a `Promise`
- `config` `<Object>` configuration object, contains config for smacker - see below
- `log` `<Object>` logging object for smacker to use, defaults to `console`. Should have `info`, `warn`, and `fatal`.

*smacker* will call `service.start` on `smacker.start`, and expect the resulting promise to resolve. It installs handlers for `SIGINT and SIGTERM` and `SIGUSR2` and calls `service.stop` when it receives one of these signals.

Unhandled exceptions and unhandled promise rejections are caught, logged, and your process will be terminated with exit code 1.

It also ensures warnings from `process.emitWarning` are logged through your logging object.


#### config

- `logJson` `<Boolean>` determines whether smacker will try to serialize the object before giving it to your logging object. Can be set via the `LOGJSON` environment variable. Defaults to `false`.
- `terminateOnMultipleResolves` `<Boolean>` smacker can terminate on the `multipleResolves` event. This is not always [desireable](https://github.com/nodejs/node/issues/24321). It defaults to `true`, since that is [recommended behaviour](https://nodejs.org/api/process.html#process_event_multipleresolves).


## Planned features

- hard shutdown if `service.start` doesn't resolve within a configurable amount of time
- hard shutdown if `service.stop` doesn't resolve within a configurable amount of time
- allow configuration `SIGUSR2` to be passed to the service for it to handle
- detecting if `service.stop` actually leaves the event loop empty  
