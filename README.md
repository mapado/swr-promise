# swr-promise

Implement an equivalent of Cache-Control [stale-while-revalidate](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cache-Control#stale-while-revalidate) directive:

This package does cache the result of a promise and keep it in cache for a given `maxAge`. If the promise is called once again, then the cache value is returned. If the cache is stale, then the cache is still served during the `swr` time, but the promise is called in background.

## Install

```bash
$ npm install @mapado/swr-promise
```

## Usage

```typescript
import swrPromise from "@mapado/swr-promise";

const fetchData = (url: string) => fetch(url);

const fiveMinute  = 5 * 60 * 1000

const fetchDataSWR = swrPromise(fetchData,{ maxAge: fiveMinute });

fetchDataSWR("https://api.example.com/data")
  .then(console.log)
  .catch(console.error);
```

## API

### swrPromise(promiseFn, [options])

#### Parameters

- `promiseFn`: The Promise function to be wrapped.
- `options`(optional): An options object with the following properties:
  - `maxAge`: Cache validity period (ms), default is 0, when it is Infinity, it is cached permanently
  - `swr`: Cache expiration tolerance time (ms), default Infinity (`stale-while-revalidate`)
  - `sie`: Update error tolerance time (ms), default is Infinity (`stale-if-error`)
  - `gcThrottle`: Garbage collection throttling time (ms), the default is 0, when it is 0, no garbage collection is performed
  - `cacheFulfilled`: Whether to cache the current normal result, the default is true (arguments, value) => boolean
  - `cacheRejected`: Whether to cache the current exception result, the default is false (arguments, error) => boolean
  - `argsEqual`(optional): Compares the two argument arrays for equality to ensure correct matching and lookup of cache entries in the cache
  - `storeCreator`(optional): creates a Map to store cached data, but you can use other media such as SQlite by passing a custom storeCreator function

#### Returns

- A new function that accepts the same arguments as `promiseFn` and returns a Promise.

## Example

```typescript
import swrPromise from "@mapado/swr-promise";

const mock = () =>
  new Promise((resolve) => setTimeout(() => resolve(Math.random()), 1000));

const swrMock = swrPromise(mock, { maxAge: 1000, swr: 1000 });

await swrMock().then(console.log); // 0.9504613827463109
await swrMock().then(console.log); // 0.9504613827463109 same response

// simulate waiting for 1 second
await sleep(1000);

await swrMock().then(console.log); // 0.9504613827463109 same response, but mock function re-executes

```

## LICENSE

This project is licensed under the MIT License.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Acknowledgements

This utility function uses:
- `concur-promise` to prevents duplicate execution of identical Promise calls
- `lodash.isequal` to compare the arguments
- `lodash.throttle` to limit the frequency of garbage collection operations.
