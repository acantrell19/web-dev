# Web Development Class 5/3/19 - Music App

## Hello and welcome for Web Development!

#### Code along in this repo has been adopted from https://levelup.gitconnected.com/how-to-build-a-spotify-player-with-react-in-15-minutes-7e01991bc4b6

First thing first, let's make a repo. 

```js
mkdir web-dev-lesson
cd web-dev-lesson
npx create-react-app music-app // this will take a while
npm start
```

Thoughout the project make sure to keep note of important things like a description of your app and directions on how to start up your frontend and backend in the `README.md`. You can look at an example [here](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2). 

We are going to develop a music app very similar to Spotify. To do let's look at the pieces that we will need: 

1. MVP - Minimal Viable Product
  * This is super important in development because often people overshoot their development estimates and try to add too many features, leading to missed deadline!
  * Ask yourself, what is the purpose of the app you are building? What are the minimal features your product must be able to do to meet that purpose? This is often provided by a Product Owner (PO) a.k.a. business counterpart.
  * Write user stories  

2. Draw up wireframes of what your app will look like. Do you have multiple views? Buttons/links that can be triggered? See good example [here](https://www.lucidchart.com/pages/templates/wireframe/facebook-wireframe-template). 

3. Git is your best friend! Make sure to commit often and push up your work. ALWAYS PULL BEFORE YOU PUSH! This is especially important when you work with others of the same branch or branch off of a `master` branch that is updated. Here is an amazing [cheat sheet](https://education.github.com/git-cheat-sheet-education.pdf).  

  * Create repo in GitHub: https://github.kdc.capitalone.com/new. Make sure it has the same name as the repo you created locally. 

```js
git init
git remote add origin https://github.com/pogrebnav17/music-app.git
git status
git add .
git commit -m "set up repo, added readme files"
git push -u origin master
```


3. Let's spin up a React app! Check out React docs [here](https://reactjs.org/docs/getting-started.html).

`npx create-react-app music-app`


4. Let's checkout our API
https://developer.spotify.com/documentation/web-api/

* Register app with Spotify: https://developer.spotify.com/dashboard/tos-accept 

* Click on edit and add `http://localhost:3000` to Redirect URIs. 


5. Add these imports 

```js
import React, { Component } from 'react';
import hash from './hash';
export const authEndpoint = 'https://accounts.spotify.com/authorize';
```

Add `clientId`, `redirectUri`, and `scopes`.

```js
// Replace with your app's client ID, redirect URI and desired scopes
const clientId = "ADD YOURS HERE";
const redirectUri = "http://localhost:3000";
const scopes = [
  "user-read-currently-playing",
  "user-read-playback-state",
];
```

Add the hashing 

```js
// Get the hash of the url
const hash = window.location.hash
  .substring(1)
  .split("&")
  .reduce(function(initial, item) {
    if (item) {
      var parts = item.split("=");
      initial[parts[0]] = decodeURIComponent(parts[1]);
    }
    return initial;
  }, {});
window.location.hash = "";
```

Delete <p> and <a> tag contents. 

6. Let's add a link <a> for the "Login to Spotify link. This is what will log us in and authenticate our API calls. 

```js
{!this.state.token && (
    <a
    className="btn btn--loginApp-link"
    href={`${authEndpoint}?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scopes.join(
      "%20"
    )}&response_type=token&show_dialog=true`}
    >
      Login to Spotify
    </a>
)}
```


7. Let's add a player component `<Player>` 

```js 
{this.state.token && (
  // Spotify Player Will Go Here In the Next Step
  )}
```

Create a Player.js. Add import up top.

```js 
import React from 'react';
import './Player.css';

// functional component 
const Player = props => {
  return (
    <div className="Player">
      Hello
      ${props}
    </div>
  )
}

export default Player;
```

8. Create constructor in `App.js` to set up/initiate state for objects from our API call 

```js 
constructor() {
    super();
    
    this.state = {
      token: null,
    item: {
      album: {
        images: [{ url: "" }]
      },
      name: "",
      artists: [{ name: "" }],
      duration_ms:0,
    },
    is_playing: "Paused",
    progress_ms: 0
  };

  this.getCurrentlyPlaying = this.getCurrentlyPlaying.bind(this);
  }
  ```

  Add `componentDidMount` to set the token after render

  ```js
  componentDidMount() {
    // Set token
    let _token = hash.access_token;

    if (_token) {
      // Set token
      this.setState({
        token: _token
      });
      this.getCurrentlyPlaying(_token);
    }
  }
  ```

  9. Write an API call (using ajax)

```js
getCurrentlyPlaying(token) {
  // Make a call using the token
  $.ajax({
    url: "https://api.spotify.com/v1/me/player",
    type: "GET",
    beforeSend: (xhr) => {
      xhr.setRequestHeader("Authorization", "Bearer " + token);
    },
    success: (data) => {
      console.log("data", data);
      this.setState({
        item: data.item,
        is_playing: data.is_playing,
        progress_ms: data.progress_ms,
      });
    }
  });
}
```

10. Pass objects to `Player` as props

```js
<Player
  item={this.state.item}
  is_playing={this.state.is_playing}
  progress_ms={this.progress_ms}
/>
```

11. Let's clean up our folder structure a little 

-src
--components
---App.js
---Player.js
--style
---App.css
---Player.css
--test
---App.test.js

12. Let add Player content 

```js
const backgroundStyles = {
    backgroundImage:`url(${props.item.album.images[0].url})`,
  };
  
  const progressBarStyles = {
    width: (props.progress_ms * 100 / props.item.duration_ms) + '%'
  };
  
  return (
    <div className="App">
      <div className="main-wrapper">
        <div className="now-playing__img">
          <img src={props.item.album.images[0].url} />
        </div>
        <div className="now-playing__side">
          <div className="now-playing__name">{props.item.name}</div>
          <div className="now-playing__artist">
            {props.item.artists[0].name}
          </div>
          <div className="now-playing__status">
            {props.is_playing ? "Playing" : "Paused"}
          </div>
          <div className="progress">
            <div
              className="progress__bar"
              style={progressBarStyles}
            />
          </div>
        </div>
        <div className="background" style={backgroundStyles} />{" "}
      </div>
    </div>
  );
```

13. Add basic styling to `Player`

```css
/** Now Playing **/
.now-playing__name {
  font-size: 1.5em;
  margin-bottom: 0.2em;
}
.now-playing__artist {
  margin-bottom: 1em;
}
.now-playing__status {
  margin-bottom: 1em;
}
.now-playing__img {
  float: left;
  margin-right: 10px;
  text-align: right;
  width: 45%;
}
.now-playing__img img {
  max-width: 80vmin;
  width: 100%;
}
.now-playing__side {
  margin-left: 5%;
  width: 45%;
}
/** Progress **/
.progress {
  border: 1px solid #eee;
  height: 6px;
  border-radius: 3px;
  overflow: hidden;
}
.progress__bar {
  background-color: #eee;
  height: 4px;
}
```

### Other super helpful Web Dev Tools: 

* Developer console a.k.a. life saver 
* Postman - test out API calls 
* CORS plugin 
* Toggle Pesticide plugin - super simple frontend tool that outlines elements in the DOM 
* [Status Code Cats](https://http.cat/)
