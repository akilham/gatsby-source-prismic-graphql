# gatsby-source-prismic-graphql

Source data from Prismic with GraphQL

### Differences from `gatsby-source-prismic`

This plugin will require [graphql enabled](https://prismic.io/blog/graphql-api-alpha-release) in your Prismic instance.

The feature is currently in _alpha_ and not recommended in production. However that being said, by using Gatsby you have the garantee of production builds to never break as they are
statically compiled.

## Installing

Install module

```bash
npm install --save gatsby-source-prismic-graphql
```

Add plugin to `gatsby-config.js`:

```js
{
  resolve: 'gatsby-source-prismic-graphql',
  options: {
    repositoryName: 'gatsby-source-prismic-test-site', // (required)
    accessToken: '...', // (optional)
    path: '/preview', // (optional, default: /preview)
    previews: true, // (optional, default: false)
    pages: [{ // (optional)
      type: 'Article',         // TypeName from prismic
      match: '/article/:uid',  // Pages will be generated under this pattern (optional)
      path: '/article',        // Placeholder page for unpublished documents
      component: require.resolve('./src/templates/article.js'),
    }],
  }
}
```

Edit your `gatsby-browser.js`:

```js
const { registerLinkResolver } = require('gatsby-source-prismic-graphql');
const { linkResolver } = require('./src/utils/linkResolver');

registerLinkResolver(linkResolver);
```

## Usage

### Fetch data from Prismic

It is very easy to fetch data from prismic.

```jsx
import React from 'react';
import { RichText } from 'prismic-reactjs';

export const query = graphql`
  {
    prismic {
      page(uid:"homepage", lang:"en-us") {
        title
        description
      }
    }
  }
`

export default function Page({ data }) => <>
  <h1>{RichText.render(data.prismic.title)}</h1>
  <h2>{RichText.render(data.prismic.description)}</h1>
</>
```

### Prismic Previews

Previews are enabled by default.

[Read how to enable previews](https://user-guides.prismic.io/preview/how-to-set-up-a-preview/how-to-set-up-a-preview) in your prismic instance.

### Generated pages

You can generate pages automatically by providing mapping configuration under the `pages` option.

If you have two blog posts like `foo`, `bar`, it will generate the following URLs:

- /blogpost/foo
- /blogpost/bar

If you create a new unpublished blogpost, `baz` it will be accessible for preview under:

- /blogpost?uid=baz

[See the example](https://github.com/birkir/gatsby-source-prismic-graphql/tree/master/examples/default)

```js
{
  pages: [{
    type: 'Article',
    match: '/blogpost/:uid',
    path: '/blogpost',
    component: require.resolve('./src/templates/article.js'),
  }],
}
```

### StaticQuery

You can use static queries like normal, but if you would like to preview them, use the `withPreview` function.

[See the example](https://github.com/birkir/gatsby-source-prismic-graphql/tree/master/examples/static-query)

```js
import { StaticQuery, graphql } from 'gatsby';
import { withPreview } from 'gatsby-source-prismic-graphql';

const articlesQuery = graphql`
  query {
    prismic {
      ...
    }
  }
`;

export const Articles = () => (
  <StaticQuery
    query={articlesQuery}
    render={withPreview(data => { ... }, articlesQuery)}
  />
);
```

### useStaticQuery

No support yet.

### Fragments

Fragments are supported for both page queries and static queries.

[See the example](https://github.com/birkir/gatsby-source-prismic-graphql/tree/master/examples/fragments)

**Page components**:

```jsx
import { graphql } from 'gatsby';

const fragmentX = graphql` fragment X on Y { ... } `;

export const query = graphql`
  query {
    ...X
  }
`;

const MyPage = (data) => { ... };
MyPage.fragments = [fragmentX];

export default MyPage;
```

**StaticQuery**:

```jsx
import { StaticQuery, graphql } from 'gatsby';
import { withPreview } from 'gatsby-source-prismic-graphql';

const fragmentX = graphql` fragment X on Y { ... } `;

export const query = graphql`
  query {
    ...X
  }
`;

export default () => (
  <StaticQuery
    query={query}
    render={withPreview(data => { ... }, query, [fragmentX])}
  />
);

```

### Pagination and other dynamic fetching

You can use this plugin to dynamically fetch different component for your component. This is great for cases like pagination. See the following example:

[See the example](https://github.com/birkir/gatsby-source-prismic-graphql/tree/master/examples/pagination)

```jsx
import React from 'react';
import { graphql } from 'gatsby';

export const query = graphql`
  query Example($limit: Int) {
    prismic {
      allArticles(first: $limit) {
        edges {
          node {
            title
          }
        }
      }
    }
  }
`;

export default function Example({ data, prismic }) {
  const handleClick = () =>
    prismic.load({
      variables: { limit: 100 },
      query, // (optional)
      fragments: [], // (optional)
    });

  return (
    // ... data
    <button onClick={handleClick}>load more</button>
  );
}
```

## How this plugin works

1. The plugin creates a new page at `/preview` (by default, you can change this), that will be your preview URL you setup in the Prismic admin interface.

   It will automatically set cookies based on the query parameters and attempt to find the correct page to redirect to with your linkResolver.

2. It uses a different `babel-plugin-remove-graphql-queries` on the client.

   The modified plugin emits your graphql queries as string so they can be read and re-used on the client side by the plugin.

3. Once redirected to a page with the content, everything will load normally.

   In the background, the plugin takes your original gatsby graphql query, extracts the prismic subquery and uses it to make a graphql request to Prismic with a preview reference.

   Once data is received, it will update the `data` prop with merged data from Prismic preview and re-render the component.

## Development

```bash
git clone git@github.com:birkir/gatsby-source-prismic-graphql.git
cd gatsby-source-prismic-graphql
yarn install
yarn setup
yarn start

# select example to work with
cd examples/default
yarn start
```

## Issues and Troubleshooting

This plugin does not have gatsby-plugin-sharp support.

Please raise an issue on GitHub if you have any problems.

### My page graphql query does not hot-reload for previews

This is a Gatsby limitation. You can bypass this limitation by adding the following:

```jsx
export const query = graphql` ... `;
const MyPage = () => { ... };

MyPage.query = query; // <-- set the query manually to allow hot-reload.
```
