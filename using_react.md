# Server
The demo server for this example code has three routes to read, add, and update todos that are stored in memory. Make sure this file is placecd and run in the top-level of your React project.

```js
const express  = require( 'express' ),
      app      = express(),
      bp       = require( 'body-parser')

const todos = [
  { name:'buy groceries', completed:false }
]

app.use( bp.json() )
app.use( express.static( 'build' ) )

app.get( '/read', ( req, res ) => res.json( todos ) )

app.post( '/add', ( req,res ) => {
  todos.push( req.body )
  res.json( todos )
})

app.post( '/change', function( req,res ) {
  const idx = todos.findIndex( v => v.name === req.body.name )
  todos[ idx ].completed = req.body.completed
  
  res.sendStatus( 200 )
})

app.listen( 8080 )
```

If you watched the Svelte tutorial, note that here we serve our files from the `build` directory, which is where `create-react-app` places all files for deployment.

# React
In this tutorial, we'll use `create-react-app` to make a new project using [the instructions provided by Facebook](https://github.com/facebook/create-react-app/). Alternatively, ou can create a basic React project (using Babel and Parcel) by following [these instructions](https://createapp.dev/parcel)

First, we require a basic file that loads our `App` component into a particular DOM element (index.js). Below is the file given by `create-react-app`; you shouldn't need to modify this file.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(<App />, document.getElementById('root'));

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

Next we create our `App.js` file. This file will be converted into valid JS by Babel, but React enables us to author
our components by freely mixing HTML and JS. 

```js
import React from 'react';
import './App.css';

// we could place this Todo component in a separate file, but it's
// small enough to alternatively just include it in our App.js file.

class Todo extends React.Component {
  // our .render() method creates a block of HTML using the .jsx format
  render() {
    return <li>{this.props.name} : 
      <input type="checkbox" defaultChecked={this.props.completed} name2={this.props.name} onChange={ e => this.change(e) }/>
    </li>
  }
  // call this method when the checkbox for this component is clicked
  change(e) {
    this.props.onclick( this.props.name, e.target.checked )
  }
}

// main component
class App extends React.Component {
  constructor( props ) {
    super( props )
    // initialize our state
    this.state = { todos:[] }
    this.load()
  }

  // load in our data from the server
  load() {
    fetch( '/read', { method:'get', 'no-cors':true })
      .then( response => response.json() )
      .then( json => {
         this.setState({ todos:json }) 
      })
  }

  // render component HTML using JSX 
  render() {
    return (
      <div className="App">
      <input type='text' /><button onClick={ e => this.add( e )}>add</button>
        <ul>
          { this.state.todos.map( (todo,i) => <Todo key={i} name={todo.name} completed={todo.completed} onclick={ this.toggle } /> ) }
       </ul> 
      </div>
    )
  }
}

export default App;
```

There's a lot of boilerplate to get started, but here we're creating two different components. The first is an item to represent a single Todo, while the second is our higher-level application. Note that we pass data from our app component to each todo via the `props` object, which lets us pass data through HTML attributes. So, in our example above, each `Todo` is created with a `name` attribute, and that attribute is subsequently available in the `props` object of the associated `Todo`. You can see we also assign/access the `.completed` and `.onclick` property of each `Todo` in the same way.

OK, last but not least we add functions or updaing our exising todos and adding new todos.

```js
  // when an Todo is toggled, send data to server
  toggle( name, completed ) {
    fetch( '/change', {
      method:'POST',
      body: JSON.stringify({ name, completed }),
      headers: { 'Content-Type': 'application/json' }
    })
  }
 
  // add a new todo list
  add( evt ) {
    const value = document.querySelector('input').value

    fetch( '/add', { 
      method:'POST',
      body: JSON.stringify({ name:value, completed:false }),
      headers: { 'Content-Type': 'application/json' }
    })
    .then( response => response.json() )
    .then( json => {
       this.setState({ todos:json }) 
    })
  }
```
