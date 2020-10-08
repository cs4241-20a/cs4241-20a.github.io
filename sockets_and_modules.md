# Socket Servers, Browserify, Webpack
## Quick refresher on Svelte
- Declarative reactive programming
- Changes to variables should trigger changes to UI according to component templates
- Simple button changing script to run at https://svelte.dev/tutorial

```js
<script>
	let loggedIn = false
	
	const login = function() {
	  // assignment triggers UI update
	  loggedIn = !loggedIn
	}
</script>

{#if loggedIn === true }
  <button on:click={login}>log out</button>
{:else}
  <button on:click={login}>log in</button>
{/if}
```

## Socket Servers w/ Svelte (realtime chat)
[ws](https://www.npmjs.com/package/ws) - A node package for creating servers that understand the WebSocket protocol

```js
const express = require('express'),
      app     = express(),
      ws      = require('ws'),
      http    = require('http')

app.use( express.static('public') )

const server = http.createServer( app )

const socketServer = new ws.Server({ server })

const clients = []

socketServer.on( 'connection', client => {
  // when the server receives a message from this client...
  client.on( 'message', msg => {
	  // send msg to every client EXCEPT
    // the one who originally sent it
    clients.forEach( c => {
      if( c !== client )
        c.send( msg )
    })
  })

  // add clien to client list
  clients.push( client )
})

server.listen( 3000 )
```

### Svelte Client

```js
<script>
  let ws, msgs = []
  window.onload = function() {
    ws = new WebSocket( 'ws://127.0.0.1:3000' )
    // when connection is established...
    ws.onopen = () => {
		ws.send( 'a new client has connected.' )
      ws.onmessage = msg => {
		  // add message to end of msgs array,
		  // re-assign to trigger UI update
        msgs = msgs.concat([ msg.data ])
      }
    }
  }

  const send = function() {
    const txt = document.querySelector('input').value
    ws.send( txt )
	  // re-assigning to msgs variable triggers UI update
    msgs = msgs.concat([ txt ])
  }
</script>

<input type='text' on:change={send} />

{#each msgs as msg }
  <h3>{msg}</h3>
{/each}
```

## Parcel vs. Webpack vs. Browserify vs. Rollup vs. …
- [**Parcel**](https://parceljs.org) - looks at your index.html file and figures out everything it needs to bundle. Can handle HTML / CSS / JS / TypeScript / Babel without needing a separate config file. Faster bundling than Webpack 
- [**Webpack**](https://webpack.js.org) - requires configuration file for all but the most trivial build setups. Able to handle more complex builds.
- [**Browserify**](http://browserify.org) + [**Gulp**](https://gulpjs.com): Requires detailed careful configuration. But you can basically do whatever you want with this combo… no build process is too complex. Run unit tests, copy build to multiple destinations on your computer, anything that Node.js is capable of doing. This tends to be my preferred option, but only because I’m used to using it. It takes a lot of time to get used to using it.

I found [this comparison between Parcel and Webpack](https://blog.jakoblind.no/parcel-webpack/) to be useful reading.

### Using Parcel

1. Make a new project directory and run `npm i parcel --save` in it.
2. Create a `index.html` file and set it up to import a .js file:
```html
<!doctype html>
<html lang='en'>
  <head>
    <script src='index.js'></script>
  </head>
  <body>
    test
  </body>
</html>
```
3.  In your index.js file, go ahead and import another file, `main.js` and print the results:
```js
import test from './main.js'

window.onload = function() {
  document.body.innerHTML = `<h1>${test}</h1>`
}
```
4. In your main.js file, export some text to print:
```js
const test = 'this is a test'

export default test
```
5. Run parcel on your index.html file: `npx parcel index.html`. This will start a server running on `localhost:1234`, visit it to see the results.
6. Look inside the `/dist` folder Parcel creates to get a sense of what it’s doing.

### Using Browserify / Watchify
1. You can’t use ES6 import/export statements without a little bit of extra work (using the babelify transform). Instead we use node.js style `require()` statements and define `module.exports`.
2. Let’s make a simple page like we did for the Parcel example: 
```html
<!doctype html>
<html lang='en'>
  <head>
    <script src='bundle.js'></script>
  </head>
  <body>
    test
  </body>
</html>
```
3. `bundle.js` will be the file that Browserify creates for you. We’ll tell Browserify to create this file by pointing it to `index.js` . Browserify will then resolve the necessary dependencies.
```js
const test = require( './main.js' )

window.onload = function() {
  document.body.innerHTML = `<h1>${test}</h1>`
}
```
4. In your main.js file, export some text to print:
```js
const test = 'this is a test'

module.exports = test
```
5. Install and run Browserify `npm i browserify` and then `npx browserify index.js -o bundle.js`
6. Start a server or just open the index.html file by double-clicking on it.
7. For continuous development (automatically recompile after changes to code) just run `npx watchify index.js -o bundle.js` instead of browserify.
