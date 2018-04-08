# node-royalroadl-api

A small, __unofficial__ fiction data API for [roalroadl.com](https://royalroadl.com), 
written in TypeScript.

All HTML is parsed using [cheerio](https://github.com/cheeriojs/cheerio), relative 
dates are converted to absolute ones using [date.js](https://github.com/matthewmueller/date).

This __will__ break in the future as the public pages are subject to change at any time, 
but since cheerio is very forgiving it is likely that parts will remain functional.

```
npm i -s node-royalroadl-api
```

```javascript
const { RoyalRoadAPI } = require('royalroadl-api');

const api = new RoyalRoadAPI();

(async () => {
  const popular = await api.fictions.getPopular();
  const titles = popular.slice(10).map((fic) => fic.title);

  console.log(`The top 10 popular fictions are: ${titles.join(', ')}`);
})();
```

## Properties

- `fiction` - an instance of the [FictionService](#fictionservice).
- `fictions` - an instance of the [FictionsService](#fictionsservice).

## FictionService

All methods of the FictionService return a Promise resolving to a 
`Fiction`, which looks like this:

```typescript
interface Fiction {
  type: string,
  title: string,
  image: string,
  status: string,
  tags: string[],
  warnings: string[],
  description: string,
  stats: FictionStats,
  author: FictionAuthor,
}
```

`FictionStats` and `FictionAuthor` are seperated for readability, and structured 
as follows.

```typescript
interface FictionStats {
  pages: number,
  ratings: number,
  favorites: number,
  followers: number,
  views: {
    total: number,
    average: number,
  },
  score: {
    style: number,
    story: number,
    grammar: number,
    overall: number,
    character: number,
  },
}

interface FictionAuthor {
  id: number,
  name: string,
  title: string,
  avatar: string,
}
```

All scores are accurate to two decimal places, and range from 0 to 5. The ID 
of an author can be used to get their profile URL by inserting it into 
`royalroadl.com/profile/${id}`.

### Methods

_A note for those unfamiliar with typescripts syntax, Promise\<Fiction\> describes 
a Promise, resolving to a `Fiction`._

- `getFiction(id: number): Promise<Fiction>` - returns the fiction at `royalroadl.com/fiction/${id}`.
- `getRandom(): Promise<Fiction>` - equivalent of `royalroadl.com/fiction/random`, AKA 'Surprise me!'.

## FictionsService

Most methods of this service return a Promise resolving to some extension of 
`FictionBlurb`, which contains the overlapping information provided by the 
active-popular, latest-updates, and best-rated pages.

```typescript
interface FictionBlurb {
  id: number,
  type: string,
  title: string,
  image: string,
  tags: string[],
}
```

Here, type is one of 'Original' or 'Fanfiction'.

Since all individual pages have their own set of extra information, they all have a 
fitting interface which extends the common base.

```typescript
interface LatestBlurb extends FictionBlurb {
  latest: {
    name: string,
    link: string,
    created: number,
  }[],
}

interface PopularBlurb extends FictionBlurb {
  description: string,
  stats: {
    pages: number,
    latest: number,
    rating: number,
    chapters: number,
    followers: number,
  },
}

interface BestBlurb extends PopularBlurb {}
```

Since the best-rated and active-popular pages have the same format, the same interface 
structure is used.

Note that the rating returned here is significantly more accurate than those 
in `Fiction`.

The `search` method returns a more limited set of information, which does not 
extend the `FictionBlurb` common base. It is described as

```typescript
interface SearchBlurb {
  id: number,
  pages: number,
  title: string,
  image: string,
  author: string,
  description: string,
}
```

### Methods

_For those unfamiliar with ES6 default parameters, read more [on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)._

- `getLatest(page: number = 1): Promise<LatestBlurb>`
- `getPopular(page: number = 1): Promise<PopularBlurb>`
- `getBest(page: number = 1): Promise<BestBlurb>`
- `search(keyword: string, page: number = 1): Promise<SearchBlurb>`