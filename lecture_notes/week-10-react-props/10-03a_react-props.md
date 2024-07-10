# Using React Props to Refactor a Form and List:

[Starter Repo](https://github.com/PrimeAcademy/react-props-starter)


## Topics

- Practice extracting components using props
- Destructuring props
- Passing callback functions as props
- Working with form components
  

Uses the Mythical Creatures App with a List and Form components. Goals:

- `if` a review about props is needed
  - make a header and pass it props
- move list to a component
- move individual list items to a component
- move form to a component

----

Our `App.js` is getting a little crowded. We should probably break these out into components now:

```
index
+-------------------------+
|App                      |
|                         |
|+---------------------+  |
||Header               |  |
|+---------------------+  |
|+---------------------+  |
||Form Input & Button  |  |
|+---------------------+  |
|                         |
|+----------------------+ |
||List of things        | |
||+--------------------+| |
|||List Item           || |
||+--------------------+| |
||+--------------------+| |
|||List Item           || |
||+--------------------+| |
|+----------------------+ |
|                         |
+-------------------------+
```

## Props
Remember: *Components are functions.* Functions can take in arguments. In React, props are merely the way that we can provide arguments to our components.

<details>
  <summary>Super simple example of props if a review is needed:</summary>

  ### Header
  Lets start by making a new  file `components/Header/Header.js`

  ```JSX

  function Header (props) {
      console.log(props)
      return (
          <>
              <h1>Hi!</h1>
          </>
      );
    }

  export default Header;
  ```

  And in `App.js` we need to import:
  ```JSX
  import Header from '../Header/Header';
  ```

  And we need to render the component:
  ```JSX
  <Header />
  ```
  So to use props, we will pass data from the parent (App) to the child (Header). Props (just like function parameters!) can be named anything you want!

  ```JSX
  <Header myProp={"YAY"} isCool={true} title={"Hello Props!"} />
  ```

  Check out your console log from header -- what do you see?

  Let's render the title prop in our header:

  ```js
  return (
          <>
              <h1>{props.title}</h1>
          </>
      );
  ```


</details>

### CreatureList

Often with props, we share existing variables! Let's make a list!

Create a new file `components/CreatureList/CreatureList.js` 

```JSX

function CreatureList(props) {

    return (
      <>
        <h2>All Creatures</h2>
        <ul>
          {
            props.creatureList.map((creature) => (
              <li key={creature.id}>
                The {creature.name} originated in {creature.origin}.
              </li>
            ))
          }
        </ul>
      </>
    );
  

export default CreatureList;
```

And in `App.js` we need to import:
```JSX
import CreatureList from '../CreatureList/CreatureList';
```

And we need to pass the correct props:
```JSX
<CreatureList creatureList={creatureList}/>
```

### Destructuring Props

It's getting annoying to always have to type `props.thing`, and we can make it a little simpler. If we `destructure` our props, we can get access to the stuff directly. You've already done this, actually:

`import {useState} from 'react` -- useState is being destructured out of react. Its a shortcut for `react.useState`

So, if we know the name of the prop we want, we can just grab it by its name!

`function CreatureList({creatureList}) {`

This is extra groovy, because now our components *communicate* to us what props (or arguments) they expect to receive.

### CreatureItem

Look at that list! It's great! But, the `<li>` itself might be a good candidate for a component, so we can make it happen with props!
Create a new file `components/CreatureItem/CreatureItem.js` 


```JSX

function CreatureItem({creature}) {

    //This component could have local state too!
    return (
      <li>
        The {creature.name} originated in {creature.origin}.
      </li>
    );
  

export default CreatureItem;
```

And use in the `.map` in CreatureList:

```JSX
  return (
    <ul>
      {
        creatureList.map((creature) => (
          <CreatureItem key={creature.id} creature={creature} />
        ))
      }
    </ul>
  );
```

Note, `key={creature.id}` and `creature={creature}` look like they are doing the same thing, but they are not.
`key={creature.id}` is assigning a unique key (label) to each CreatureItem component that's retuned, so react can keep track of it. Things like `key` and `className` are special "keyword" properties on a React component that won't passed down as props.


### CreatureForm

Now the tricky one...

Create a new file `components/CreatureForm/CreatureForm.js`.

Then, let's move over the JSX code for the form, the React state that the form's inputs rely on, and the `createCreature` function that gets called when the form is submitted:

```JSX

function CreatureForm() {
    const [newCreatureName, setNewCreatureName] = useState('')
    const [newCreatureOrigin, setNewCreatureOrigin] = useState('')
    
    // Function to add a new creature to the database:
    const createCreature = (event) => {
      event.preventDefault();

      axios({
        method: 'POST',
        url: '/api/creatures',
        data: {
          name: newCreatureName,
          origin: newCreatureOrigin
        }
      })
        .then((response) => {
          // Call fetchCreatures to bring our React app's state back
          // "in sync" with the underlying creatures table:
          fetchCreatures(); // 👈 THIS FUNCTION DOESN'T EXIST
                            //    IN THIS COMPONENT!! SHOOT!!
                            //    UH. NOW WHAT!?

          // Clear the form inputs:
          setNewCreatureName('');
          setNewCreatureOrigin('')
        })
        .catch((error) => {
          console.log('Error on add:', error);
        });
    }
    
    return (
      <form onSubmit={createCreature}>
        <label>Name:</label>
        <input 
          onChange={(event) => setNewCreatureName(event.target.value)} 
          value={newCreatureName}/>
        <label>Origin:</label>
        <input 
          onChange={(event) => setNewCreatureOrigin(event.target.value)} 
          value={newCreatureOrigin}/>
        <button type="submit">Add New Creature</button>
      </form>
    );
  }


export default CreatureForm;
```


This `CreatureForm` component has everything it needs to "do its job," **except for one very important detail**. It's going to throw an error, because the `fetchCreatures` function does not exist within the scope of this component.

Think of the `createCreature` function as a machine that will always send a POST request to '/creatures', then run the `fetchCreatures` function. To do its job, it **depends** upon the `fetchCreatures` function. This is called a **dependency**.

We could technically just copy and paste the entire `fetchCreatures` function into this component. That would totally work. Duplicating code is not a happy path to go down if we want a codebase that is straightforward to maintain. (Imagine if we copy and pasted it three more times. Then, later, someone asks us to change some detail of how that function works... We'd have lots of places within our codebase that we'd be on the hook to modify.)

Instead of copy and pasting, we can **pass a refernce to the function** into our `CreatureForm` component as a prop! This is a big concept to wrap your head around. This is, like, **the** React concept that we need to wrap our heads around, though. If a given component depends on state or functionality that exists in another component, we have to pass it **downward** as a prop.


Here's how we'd pass a reference to this function down to `CreatureForm` as a prop:

```JSX
// In App.js
// Import the CreatureForm
import CreatureForm from '../CreatureForm/CreatureForm';

// ... 


// And render it:
<CreatureForm 
  // Gotta pass it this function:
  fetchCreatures={fetchCreatures} 
/>
```

We can also remove these bits of state from `App`, since they are now managed by the `CreatureForm`:

```js
// Get rid of these in App.jsx
// const [newCreatureName, setNewCreatureName] = useState('');
// const [newCreatureOrigin, setNewCreatureOrigin] = useState('');
```

Lastly, we need to modify `CreatureForm` to accept the `fetchCreatures` function as a prop:

```JSX
function CreatureForm({fetchCreatures}) {
    const [newCreatureName, setNewCreatureName] = useState('')
    const [newCreatureOrigin, setNewCreatureOrigin] = useState('')
    // ...
```

### Check Out App, Now!

Check out how clean `App.jsx` looks now! This is a big win. Look especially closely at App's return statement. It now more clearly communicates what it does. We don't have to read a billion lines of code to see what's happening. In human language, we can now see that App is a `<div>` that contains:
  - A `<CreatureForm>` that depends on a function called `fetchCreatures` to do its job.
  - A `<CreatureList>` that depends on a piece of state called `creatureList` to do its job.

Compared to where we started with `App.jsx`, this code does the same thing...but it takes much less time for a human to look at it and have an idea of what it does. React gives us the ability to *compose* applications that more clearly communicate what they do. It gives us the power to create abstractions that make it easier for teammates (and our future selves!) to quickly get a high-level understanding of what a given React application does.

```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';

import './App.css';

function App () {
  const [creatureList, setCreatureList] = useState([]);

  useEffect( () => {
    fetchCreatures();
  }, [])

  // Get the creatures data from the server and store it in
  // our creatureList React state:
  const fetchCreatures = () => {
    axios({
      method: 'GET',
      url: '/api/creatures'
    })
      .then((response) => {
        console.log('response.data is:', response.data);

        // Store the creatures array in React state:
        setCreatureList(response.data);
      })
      .catch((error) => {
        console.log('Error on get:', error);
      });
  }
  
  // Whoa that's 👇 clean!
  return (
    <div className="App">
      <CreatureForm fetchCreatures={fetchCreatures} />
      <CreatureList list={creatureList}/>
    </div>
  );

}

export default App
```
