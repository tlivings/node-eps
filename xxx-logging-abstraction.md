| Title  | Logging abstraction for Node.js core |
|--------|--------------------------------------|
| Author | @tlivings and @indutny               |
| Status | DRAFT                                |
| Date   | 2016-10-20                           |

# Logging abstraction for Node.js core

## Rationale

Logging in Node.js applications is often either non-existent or global (read
`console.log`). Sometimes it is provided by framework, but external modules
still may use something else.

Providing an abstraction and at least one implementation in Node.js core should
unify user-land modules efforts and simplify integration of 3rd party modules
into enterprise projects.

Furthermore, having such implementation in Node.js core will make it to the
browserified code in browser. Meaning that the same abstraction will be used in
both browsers and servers.

## Technical aspects

It is proposed to use `EventEmitter` as a base abstraction for such logging
routines. Log events may be published, subscribed with filters or without them.

## Proposed API

The core module may be named just `log`, and may be obtained by usual:

```js
const log = require('log');
```

Subscribers are global, while publishers are local. Publishers may be created
with:

```js
const p = log.createPublisher(/* optional */ name);
```

Subscribers are managed in the same way as timers:

```js
const subscriber = log.subscribe(/* optional */ filterOptions);
log.unsubscribe(subscriber);
```

**(TODO(tvlings): why?)**
All subscribers can be disconnected from publishers at any time:

```js
log.unsubscribeAll();
```

Log events are published with:

- `log(/* optional */ tags, data)` - publishes a log event with two parameters:
    - `meta` - `{ source, name, timestamp, tags }`.
    - `data` - any data passed to `log`.

Meta data in the event is as follows:

- `source` - the module name this logger is being called from.
- `name` - optional name of the logger passed to `createLogger`.
- `timestamp` - timestamp of this event.
- `tags` - optionally passed to the `log` function.

### Usage

```js
const log = require('log');

const p = log.createPublisher(/* optional name */);

p.log('debug', 'hello world.');
```

### Higher level example using RxJs to filter and format

```js
const p = log.createPublisher('test');

//Create a Rx observable publisher around the emitter.
const publisher = Rx.Observable.create((observer) => {
  const listener = log.subscribe(({ source, name, timestamp, tags }, data) => {
    observer.next({ source, name, timestamp, tags, data });
  });

  return function () {
    log.unsubscribe(listener);
  };
});

//Create a filtered and formatted event stream
const filterAndFormat = publisher.filter(
    ({ name, tags }) => {
        return name === 'test' && !!~tags.indexOf('debug'); //filter on name, module, tags, etc.
    }
).map(
    ({ source, name, timestamp, data }) => {
        return `${new Date(timestamp).toISOString()} ${source}.${name}: ${JSON.stringify(data)}`;
    }
);

const subscription = filterAndFormat.subscribe(
    (message) => {
        //Only name 'test' and tags with 'debug' will land here
        console.log(message);
    }
);

p.log('debug', 'hello world');
p.log('this will be ignored');

subscription.unsubscribe();
```
