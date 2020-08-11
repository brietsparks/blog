---
title: Easy relational React state with Normalized Reducer, Part 2
date: "2020-08-10T16:00:00Z"
---

This is Part 2 of 3 in this tutorial series about Normalized Reducer. In Part 1, I explained the usefulness of normalized data and introduced [Normalized Reducer](https://github.com/brietsparks/normalized-reducer). Then we built the first part of our bookmarks app by using Normalized Reducer to implement creation for profile entities. Here in Part 2, we will implement deleting, updating, and moving/reordering.

## Entity Deletion
We will implement deletion via [`actionCreators.delete`](https://github.com/brietsparks/normalized-reducer#delete). Make a `Profile` component that looks like:

```jsx
import IconButton from '@material-ui/core/IconButton';
import DeleteIcon from '@material-ui/icons/Delete';
// ...

function Profile({ id }) {
  const { dispatch } = useContext(ModelContext);

  const handleDelete = () => {
    dispatch(actionCreators.delete('profile', id));
  }

  return (
    <div style={styles.profile}>
      Profile ID: {id}

      <IconButton onClick={handleDelete} color="secondary"><DeleteIcon/></IconButton>
    </div>
  );
}
``` 

In the `Profiles` component (from Part 1) replace `{id}` with `<Profile id={id}/>`:
```jsx
function Profiles() {
  //
  // ...
  //

  return (
    <div>
      <Button onClick={createProfile} color="primary">New Profile</Button>

      <div style={styles.profilesInner}>
        {ids.map(id => (
          <Card key={id} style={styles.card}>
            <Profile id={id}/>
          </Card>
        ))}
      </div>
    </div>
  );
}
```

Save and reload the page. You should now be able to create and delete the items. If you `console.log` the state, when you delete a profile, you should see its data object vanish from the `entities.profile` hashmap and its ID vanish from the `ids.profile` array.

## Entity Update

Now let's display some profile attributes using [`selectors.getEntity`](https://github.com/brietsparks/normalized-reducer#getEntity), and implement attribute updates using [`actionCreators.update`](https://github.com/brietsparks/normalized-reducer#update). Add these to the `Profile` component:

```jsx
function Profile({ id, index }) {
  const { state, dispatch } = useContext(ModelContext);

  const profile = selectors.getEntity(state, { type: 'profile', id });
  if (!profile) {
    return null;
  }

  const handleChangeName = e => {
    dispatch(actionCreators.update('profile', id, { name: e.target.value }))
  }

  const handleDelete = () => {
    dispatch(actionCreators.delete('profile', id));
  }

  return (
    <div style={styles.profile}>
      <div style={styles.profileName}>
        <TextField
          autoFocus
          fullWidth
          placeholder={`Profile: (ID ${id})`}
          value={profile.name || ''}
          onChange={handleChangeName}
        />
      </div>

      <IconButton onClick={handleDelete} color="secondary"><DeleteIcon/></IconButton>
    </div>
  );
}
```

Save and reload the app. Get some profiles showing, and you should see text inputs for each. If you `console.log` the state, you should see the profile's data change as you type.

## Entity Move

Next, let's allow the user to change the order of the profiles. On each, we will add an up-button that decrements the profile's position in the collection, and a down-button that increments it. We will use [`actionCreators.move`](https://github.com/brietsparks/normalized-reducer#move) to dispatch the action.

```jsx
function Profile({ id, index }) {
  const { state, dispatch } = useContext(ModelContext);

  const profile = selectors.getEntity(state, { type: 'profile', id });
  if (!profile) {
    return null;
  }

  const handleChangeName = e => {
    dispatch(actionCreators.update('profile', id, { name: e.target.value }))
  }

  const handleMoveUp = () => {
    dispatch(actionCreators.move('profile', index, index - 1));
  }
  
  const handleMoveDown = () => {
    dispatch(actionCreators.move('profile', index, index + 1));
  }

  //
  // ...
  //

  return (
    <div style={styles.profile}>
      <div style={styles.profileMoveButtons}>
        <IconButton onClick={handleMoveUp}><UpIcon/></IconButton>
        <IconButton onClick={handleMoveDown}><DownIcon/></IconButton>
      </div>

      // ...
    </div>
  );
}
```

Not too different from the other actions. Save and reload the app, and try it out. Once again, `console.log` the state, and when you move an entity, you will see its ID in `ids.profile` get reindexed.

This wraps up Part 2! You can see the final product of Part 2 here: 
- [Part 2 source code](https://github.com/brietsparks/normalized-reducer-demo/blob/master/src/tutorial/part-2/index.js#L1)
- [Part 2 live page](https://normalized-reducer-demo.now.sh/tutorial/part-2)

In Part 3, we will implement a one-to-many relationship by allowing a user to manage bookmarks of a profile.
