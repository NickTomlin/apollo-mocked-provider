# apollo-mocked-provider

> Note: this is a temporary fork of https://github.com/benawad/apollo-mocked-provider
> for my own personal use. Please use/and/or contribute to the upstream.

Automatically mock GraphQL data with a mocked ApolloProvider

Inspiration: https://www.freecodecamp.org/news/a-new-approach-to-mocking-graphql-data-1ef49de3d491/

## install

```
yarn add apollo-mocked-provider
```

## setup

```jsx
import {
  createApolloErrorProvider,
  createApolloMockedProvider,
  createApolloLoadingProvider,
} from 'apollo-mocked-provider';
import { typeDefs } from './typeDefs';

export const ApolloMockedProvider = createApolloMockedProvider(typeDefs);
export const ApolloErrorProvider = createApolloErrorProvider();
export const ApolloLoadingProvider = createApolloLoadingProvider();
```

You can get the `typeDefs` with this helper file

```js
// downloadTypeDefs.js
const { fetchTypeDefs } = require('apollo-mocked-provider');

(() => {
  fetchTypeDefs({ uri: 'http://localhost:4000/graphql' });
})();
```

Then run that file

```
node downloadTypeDefs.js
```

## testing

```jsx
import React from 'react';
import { render, cleanup } from '@testing-library/react';
import { Todos } from './Todos';
import {
  ApolloLoadingProvider,
  ApolloErrorProvider,
  ApolloMockedProvider,
} from './test-utils/providers';

afterEach(cleanup);

test('TodoForm', async () => {
  const { debug } = render(
    <ApolloMockedProvider>
      <Todos />
    </ApolloMockedProvider>
  );

  debug();
  await Promise.resolve();
  debug();
});
```

Loading:

```jsx
import React from 'react';
import { render, cleanup } from '@testing-library/react';
import { Todos } from './Todos';
import {
  ApolloLoadingProvider,
  ApolloErrorProvider,
  ApolloMockedProvider,
} from './test-utils/providers';

afterEach(cleanup);

test('TodoForm', async () => {
  const { debug } = render(
    <ApolloLoadingProvider>
      <Todos />
    </ApolloLoadingProvider>
  );

  debug();
});
```

Error:

```jsx
import React from 'react';
import { render, cleanup } from '@testing-library/react';
import { Todos } from './Todos';
import {
  ApolloLoadingProvider,
  ApolloErrorProvider,
  ApolloMockedProvider,
} from './test-utils/providers';

afterEach(cleanup);

test('TodoForm', async () => {
  const { debug } = render(
    <ApolloErrorProvider graphQLErrors={[{ message: 'something went wrong' }]}>
      <Todos />
    </ApolloErrorProvider>
  );

  debug();
  await Promise.resolve();
  debug();
});
```

Custom mocks:

```jsx
import React from 'react';
import { render, cleanup } from '@testing-library/react';
import { Todos } from './Todos';
import {
  ApolloLoadingProvider,
  ApolloErrorProvider,
  ApolloMockedProvider,
} from './test-utils/providers';

afterEach(cleanup);

test('TodoForm', async () => {
  const { debug } = render(
    <ApolloMockedProvider
      customResolvers={{
        Query: () => ({
          todos: () => [{ id: 1, type: 'hello from custom mocked data' }],
        }),
      }}
    >
      <Todos />
    </ApolloMockedProvider>
  );

  debug();
  await Promise.resolve();
  debug();
});
```

### Cache

By default, providers will use a new instance of [`InMemoryCache`](https://www.apollographql.com/docs/react/advanced/caching/#inmemorycache), but you can override that at a global or per component level by providing an object that implements `ApolloCache` to the `create*` methods or mocked components respectively.

```jsx
import { InMemoryCache } from 'apollo-boost';

// global, shared cache
const globalCache = new InMemoryCache();
export const ApolloMockedProvider = createApolloMockedProvider(
  typeDefs,
  globalCache
);

test('local cache', async () => {
  // local, scoped cache
  const localCache = new InMemoryCache();
  const { debug } = render(
    <ApolloMockedProvider cache={localCache}>
      <Todos />
    </ApolloMockedProvider>
  );
});
```
