---
title: Easy relational React state with Normalized Reducer, Part 3
date: "2020-08-10T17:00:00Z"
---

This is Part 3 of 3 in this tutorial series about [Normalized Reducer](https://github.com/brietsparks/normalized-reducer). In Part 2, we implemented deleting, updating, and moving/reordering of profile entities. Here in Part 3, we will implement a one-to-many relationship by allowing a user to manage bookmarks of a profile.

## Attach
Let's make a `Bookmarks` component that has a button for bookmark-creation and a list of existing bookmark ID's:

```jsx
function Bookmarks({ profileId, bookmarkIds = [] }) {
  const { dispatch } = useContext(ModelContext);

  const createBookmark = () => {
    const bookmarkId = randomNumber();
    dispatch(actionCreators.create('bookmark', bookmarkId, { id: bookmarkId }));
    dispatch(actionCreators.attach('bookmark', bookmarkId, 'profileId', profileId));
  }

  return (
    <div>
      <Button onClick={createBookmark} color="primary">New Bookmark</Button>

      {bookmarkIds.map(id =>
        <div>{id}</div>
      )}
    </div>
  );
}
```

Notice the actions dispatched — we first `create` the bookmark before `attach`ing it to the profile. Reload the page, and create a profile and a bookmark. If you `console.log` the state, you will see the entities automatically be assigned each other's ID, thereby forming a relational association. Additionally, the data will have the correct cardinality; n-to-many relations will be an array of ID's, and n-to-one will be a single ID. Your state should look something like:

```json5
{
  entities: {
    profile: {
      'p1': { id: 'p1', bookmarkIds: ['b1'] },
    },
    bookmark: {
      'b1': { id: 'b1', profileId: 'p1' },
    },
  },
  ids: {
    profile: ['p1'],
    bookmark: ['b1'],
  }
}
```

## Update, move, and deletion of attached entity
Next, let's create a `Bookmark` component and implement the same functionality as we did from `Profile`. Update and delete will look the same. However, we will use [`moveAttached`](https://github.com/brietsparks/normalized-reducer#moveattached) instead of `move` because we want to reindex a bookmark in respect to the profile it belongs to.

```jsx
function Bookmark({ profileId, id, index }) {
  const { state, dispatch } = useContext(ModelContext);
  const bookmark = selectors.getEntity(state, { type: 'bookmark', id });
  if (!bookmark) {
    return null;
  }

  const handleChangeUrl = e => {
    dispatch(actionCreators.update('bookmark', id, { url: e.target.value }))
  }

  const handleMoveUp = () => {
    dispatch(actionCreators.moveAttached('profile', profileId, 'bookmarkIds', index, index - 1));
  }

  const handleMoveDown = () => {
    dispatch(actionCreators.moveAttached('profile', profileId, 'bookmarkIds', index, index + 1));
  }

  const handleDelete = () => {
    dispatch(actionCreators.delete('bookmark', id));
  }

  return (
    <div style={styles.bookmark}>
      <div style={styles.bookmarkMoveButtons}>
        <IconButton onClick={handleMoveUp} size="small">
          <UpIcon fontSize="small"/>
        </IconButton>
        <IconButton onClick={handleMoveDown} size="small">
          <DownIcon fontSize="small"/>
        </IconButton>
      </div>

      <TextField
        autoFocus
        fullWidth
        placeholder={`URL: (ID ${id})`}
        value={bookmark.url || ''}
        onChange={handleChangeUrl}
      />

      <div>
        <IconButton
          onClick={handleDelete}
          color="secondary"
          size="small"
        ><DeleteIcon fontSize="small"/></IconButton>
      </div>
    </div>
  );
}
``` 
Notice the arguments passed to `moveAttached`. The first is the type of entity whose collection we are reindexing. We are reindexing the `bookmarkIds` of a `profile`, so the argument is `'profile'`. The second argument points to the specific entity of that type, via its ID. The third is the attribute that has the reindexable collection, which is `bookmarkIds`. The last two arguments are the current index and the new index.

Give it a test run with the state logged to console. Notice that deleting a bookmark automatically removes its ID from its profile's `bookmarkIds` array.

## Cascading deletion
At this point, the user is able to do CRUD operations on entities of both types. However, since we added bookmarks, we need to implement cascading delete on profile deletion, so that when a profile is deleted, its bookmarks will be deleted too. Currently, if you delete a profile, it will vanish, but its bookmarks will remain in state, unattached and not displayed. To do a cascading delete, add the following third argument to the `handleDelete` in the `Profile` component:

```js
const handleDelete = () => {
  dispatch(actionCreators.delete('profile', id, { bookmarkIds: {} }));
}
```

The `{ bookmarkIds: {} }` tells the reducer to delete any bookmarks attached to the deletable profile. The empty object literal assigned to `bookmarkIds` means we're not cascading beyond `profile > bookmark`. However, if we had a third entity type attachable to a bookmark, such as a `tag`, we could do the following cascading delete:

```js
actionCreators.delete('profile', id, { 
  bookmarkIds: {
    tagIds: {}
  } 
})
```

In this hypothetical scenario, it would delete the profile, all bookmarks attached to the profile, and all tags attached to those bookmarks.

## Conclusion

At this point the app supports various CRUD operations on entities of a one-to-many relationship. No reducer, action, or selector logic needed be written because all that is handled by Normalized Reducer. All you had to do was define the schema and call the correct actions and selectors.

You can see the final product of Part 3 here: 
- [Part 3 source code](https://github.com/brietsparks/normalized-reducer-demo/blob/master/src/tutorial/part-3/index.js#L1)
- [Part 3 live page](https://normalized-reducer-demo.now.sh/tutorial/part-3)

We didn't cover all of Normalized Reducer's features. Here are a few: 

- action-creators: `detach`, `sort`, `sortAttached`, `setState`, and `batch`. 
- normalizr, Redux, and Redux-Toolkit integration
- many-to-many and one-to-one relationships

On that note, I challenge you to expand upon the model by adding the aforementioned `tag` entity type to the model and implementing many-to-many bookmark tagging. 

The schema would look like:  

```json5
{
  profile: {
    bookmarkIds: { type: 'bookmark', cardinality: 'many', reciprocal: 'profileId' }
  },
  bookmark: {
    profileId: { type: 'profile', cardinality: 'one', reciprocal: 'bookmarkIds' },
    tagIds: { type: 'tag', cardinality: 'many', reciprocal: 'bookmarkIds' }
  },
  tag: {
    bookmarkIds: { type: 'bookmark', cardinality: 'many', reciprocal: 'tagIds' }
  },
}
```

and the state:

```json5
{
  entities: {
    profile: {
      'p1': { bookmarkIds: ['b1'] },
      'p2': { bookmarkIds: ['b2'] }
    },
    bookmark: {
      'b1': { profileId: 'p1', tagIds: ['t1'] },
      'b2': { profileId: 'p2', tagIds: ['t1', 't2'] }
    },
    tag: {
      't1': { bookmarkIds: ['b1', 'b2'] },
      't2': { bookmarkIds: ['b2'] }
    }
  },
  ids: {
    profile: ['p1', 'p2'],
    bookmark: ['b1', 'b2'],
    tag: ['t1', 't2']
  }
}
```

Lastly, if you are deciding whether to use Normalized Reducer in project, you should also consider two well-known alternatives, [Redux-ORM](https://github.com/redux-orm/redux-orm) and [Redux-Toolkit entityAdapter](https://redux-toolkit.js.org/api/createEntityAdapter). The Normalized Reducer docs provide a comparison to these. Additionally, examine whether you actually a relational store on the frontend. The ideal use case is if you have a single-page-app or feature(s) that where frequent changes must sync across the relational data without page reloads or intermittent backend round-trips. Less-than-ideal cases would be if you have infrequent changes that are more easily synced with manual backend reconciliation, or if your app's relational model is spread across multiple pages. 
 
