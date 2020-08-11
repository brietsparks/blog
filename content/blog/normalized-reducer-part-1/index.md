---
title: Easy relational React state with Normalized Reducer, Part 1
date: "2020-08-10T15:40:32.169Z"
---

In this tutorial series, I'll show you a super easy way to manage normalized relational React state using a library called [Normalized Reducer](https://github.com/brietsparks/normalized-reducer).

## Primer on relationality and normalization

As a primer, the following is a brief what-and-why of relationality and normalization.

### The what

Data is "relational" when associations exist among data entities in ways we would describe as has-one, has-many; e.g. an author has many posts, a post has one author. Relational data is "normalized" when each entity has a single source of truth and references its related data through identifiers.

To illustrate, suppose we have posts and tags, where a post can have many tags, and a tag can be attached to multiple posts. We can represent the data in a denormalized and normalized way.

A denormalized representation would be:

```js
const posts = [
  { 
    id: 'p1',
    title: 'post 1',
    tags: [
      { id: 't1', value: 'tag 1' },
      { id: 't2', value: 'tag 2' }
    ]
  },
  {
    id: 'p2',
    title: 'post 2',
    tags: [
      { id: 't2', value: 'tag 2' },
      { id: 't3', value: 'tag 3' }
    ]
  },
]
``` 

Notice: 
- data duplication. The data of tag `t2`, which is associated with two posts, is duplicated across two places.
- relationships are represented by nesting entities within one another

A normalized representation would be:

```js
const data = {
  posts: {
    'p1': { id: 'p1', title: 'post 1', tagIds: ['t1', 't2'] },
    'p2': { id: 'p2', title: 'post 2', tagIds: ['t2', 't3'] },
  },
  tags: {
    't1': { id: 't1', value: 'tag 1', postIds: ['p1'] },
    't2': { id: 't2', value: 'tag 2', postIds: ['p1', 'p2'] },
    't3': { id: 't3', value: 'tag 3', postIds: ['p2'] },
  }
}
```

Notice:
- no data duplication. The data of tag `t2` exists in one place 
- relationships a formed by IDs which correspond to their associated entity 

### The why

Normalized data is easier to reason about. Some benefits are:
- each entity has a single source of truth, so all writes and reads of a given entity occur from one place.
- entities are not nested within other entities, so you can access any entity from a flat list instead of a deep tree.

## Using Normalized Reducer

Now the feature presentation — you can manage normalized React state with [Normalized Reducer](https://github.com/brietsparks/normalized-reducer). It abstracts common relational data access patterns into a simple API. As per the name, Normalized Reducer utilizes the reducer pattern. It is agnostic and zero-dependency, so you can use it with any reducer state implementation including React `useReducer` and Redux.

This tutorial will walk through how to build a React app with Normalized Reducer. The app will be a bookmarks manager where multiple users can bookmark URL's. The app will use Material UI for styling, but it is not required for Normalized Reducer. All you need is a working React webapp. 

In your app, install Normalized Reducer. Run:
```
yarn add normalized-reducer
```
Or
```
npm install --save normalized-reducer
```

## The Model and Store, and App
We'll start with the relational model, which has profiles and bookmarks. A profile has many bookmarks, and a bookmark has one profile. With Normalized Reducer, you define relationships in a `schema` object:

```js
const schema = {
  profile: {
    bookmarkIds: { type: 'bookmark', cardinality: 'many', reciprocal: 'profileId' }
  },
  bookmark: {
    profileId: { type: 'profile', cardinality: 'one', reciprocal: 'bookmarkIds' }
  }
}
```

Import `normalized-reducer` and pass in the schema.

```js
import makeNormalizedSlice from 'normalized-reducer';

const {
  emptyState,
  actionCreators,
  reducer,
  selectors,
  actionTypes,
} = makeNormalizedSlice(schema)
```  

The returned variables are what help you manage state in way that is consistent with your model. The variable names should look familiar if you've used reducers in React or Redux.
- the `reducer` can be fed to a React `userReducer` or Redux `createStore` or Redux `combineReducers`
- the `actionCreators` contains functions that can be called and fed into a dispatcher to change state
- `selectors` contains functions that can be fed state and will return a piece of state
- `emptyState` is the zero-value object specific to your model
- `actionTypes` contains constants for utility purposes

Now we will use `reducer` and `emptyState` to set up a context+hooks store, and a root `App` component.

```jsx
import Container from '@material-ui/core/Container';
// ...

const ModelContext = createContext();

function Store({ children }) {
  const [state, dispatch] = useReducer(reducer, emptyState);
  const value = { state, dispatch }
  return (
    <ModelContext.Provider value={value}>
      {children}
    </ModelContext.Provider>
  );
}

export default function App() {
  return (
    <Container maxWidth="sm">
      <Store>
        <div>This is a placeholder component</div>
      </Store>
    </Container>
  );
}
```

## A quick detour for styling
Next, let's get styling out of the way. You implement your own styling, or none at all, or follow these steps for predefined styling:
 
1. Copy this [styles.js](https://github.com/brietsparks/normalized-reducer-demo/blob/master/src/tutorial/styles.js) into your project. Import it, and use it whenever you see the `style` variable in this tutorial. 
2. Copy this [theme.js](https://github.com/brietsparks/normalized-reducer-demo/blob/master/src/theme.js) file into your project. Import `ThemeProvider` from Material UI and render it `App` with the imported theme.
3. Import `CssBaseline` from Material UI and render it in `App`.

It should look like:

```jsx
import { ThemeProvider } from '@material-ui/core/styles';
import CssBaseline from '@material-ui/core/CssBaseline';
import theme from './theme';
import styles from './styles';

export default function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Container maxWidth="sm" style={styles.container}>
        <Store>
          <div>This is a placeholder component</div>
        </Store>
      </Container>
    </ThemeProvider>
  );
}
```

## Entity Creation

Let's build our first bit of functionality by allowing the user to create a profile. It will display the profile IDs using the [`selectors.getIds`](https://github.com/brietsparks/normalized-reducer#getIds) selector, and it will enable profile–creation using the [`actionCreators.create`](https://github.com/brietsparks/normalized-reducer#create) action-creator. The component:

```jsx
import Button from '@material-ui/core/Button';
import Card from '@material-ui/core/Card';
// ...

function Profiles() {
  const { state, dispatch } = useContext(ModelContext);
  const ids = selectors.getIds(state, { type: 'profile' });

  const createProfile = () => {
    const id = randomNumber();
    dispatch(actionCreators.create('profile', id, { id }));
  }

  return (
    <div>
      <Button onClick={createProfile} color="primary">New Profile</Button>

      <div style={styles.profilesInner}>
        {ids.map(id => (
          <Card key={id} style={styles.card}>
            {id}
          </Card>
        ))}
      </div>
    </div>
  );
}

function randomNumber(length) {
  const n = Math.floor(Math.pow(10, length-1) + Math.random() * (Math.pow(10, length) - Math.pow(10, length-1) - 1));
  return n.toString();
}
```

Notice the arguments passed to `actionCreators.create`. The first is the type of entity. In our case this is either `'profile'` or `'bookmark'`. The second is a unique identifier. The library doesn't generate IDs, so you must provide one. The last argument, an optional one, is an object of arbitrary data.

Next, in `App`, remove the placeholder and render `<Profiles/>` instead: 

```jsx
export default function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Container maxWidth="sm" style={styles.container}>
        <Store>
          <Profiles/>
        </Store>
      </Container>
    </ThemeProvider>
  );
}
```

Load the app in your browser. Click the `New Profile` button a few times, and you should see some random profile IDs appear on the page. To witness the state changes, `console.log` the `state`:

```js
function Store({ children }) {
  const [state, dispatch] = useReducer(reducer, emptyState);
  console.log(state)

  // ...
}
```

And then have another run through, and the console will log an object that looks like:

```json5
{
  entities: {
    profile: {
      // your ids will be random numbers
      'p1': { id: 'p1' },
      'p2': { id: 'p2' },
      'p3': { id: 'p3' },
    },
    bookmark: {
    },
  },
  ids: {
    profile: ['p1', 'p2', 'p3'],
    bookmark: [],
  }
}
```

This is the normalized shape in action. The entity data collection is a flat key-to-object hashmap, and entity order is an array of ids. If you have ever worked with [normalizr](https://github.com/paularmstrong/normalizr) this will look familiar. Later, when we fill out the rest the app functionality, the state will look like:

```json5
{
  entities: {
    profile: {
      'p1': { bookmarkIds: ['b1'] },
      'p2': { bookmarkIds: ['b2'] }
    },
    bookmark: {
      'b1': { profileId: 'p1' },
      'b2': { profileId: 'p2' }
    },
  },
  ids: {
    profile: ['p1', 'p2'],
    bookmark: ['b1', 'b2'],
  }
}
```

Wrapping up Part 1. Notice that the only state management code you had to write was the schema plus a few function calls. No writing reducer logic, action boilerplate, etc. This is the gist of Normalized Reducer.
 
You can see the final product of Part 1 here: 
 - [Part 1 source code](https://github.com/brietsparks/normalized-reducer-demo/blob/master/src/tutorial/part-1/index.js#L1)
 - [Part 1 live page](https://normalized-reducer-demo.now.sh/tutorial/part-1)

In Part 2 of the tutorial, we'll continue implementing functionality, including deleting, updating, and moving/reordering entities.
