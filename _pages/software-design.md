---
layout: page
title: Software Design & Engineering
permalink: /software-design
---

- **Original Artifact:** Animal Shelter Dashboard from CS 340 from Spring 2024 
- **Enhancement:** Added real-time updates via Socket.IO, opt-in live comments

---

## Purpose of App

The dashboard is a web page that shows information about animals in a shelter: things like each animal’s name, breed, and location. Volunteers (or staff members) log in, and they can:

- See a table listing every animal  
- View a pie chart that breaks down how many of each breed there are  
- Click on a map to see where an animal is located  

When one volunteer marks an animal as “adopted” or adds a new animal, everyone else using the dashboard can see that change without refreshing their page—if they’ve chosen to **Subscribe to Live Comments**. There’s also a box below the table where volunteers can leave quick notes (for example: “I fed Bella at 10 AM”). Anyone who has checked that box will see those notes appear immediately.

> *This turns a one-user demo into a true collaborative tool.*

---

## Changes Made

- **Opt-in Live Comments**  
  Added a checkbox labeled **Subscribe to Live Comments**. When checked, the client listens on a separate Socket.IO channel (`new_comment`); when unchecked, it stops listening.  
- **Separate Data vs. Comment Channels**  
  • Animal create/update/delete events broadcast on `"new_data"`  
  • Comments broadcast on `"new_comment"`  
- **Refactored “Post Comment”**  
  Changed the button handler to emit `post_comment` (server then rebroadcasts it), instead of simply writing locally in the notebook.  
- **Minimal UI Impact**  
  Volunteers who don’t need live chatter can leave it off; busy shelters won’t get unwanted pop-ups.

---

## Important Code Snippets

### Client-Side: Toggle Live Comments

```javascript
// assets/js/app.js
const socket = io();
const checkbox = document.querySelector('#subscribe-comments');

checkbox.addEventListener('change', () => {
  if (checkbox.checked) {
    socket.on('new_comment', renderComment);
  } else {
    socket.off('new_comment', renderComment);
  }
});

function renderComment(data) {
  const list = document.querySelector('#comment-list');
  const li = document.createElement('li');
  li.textContent = `${data.user}: ${data.text}`;
  list.appendChild(li);
}

# Server-side: Emitting Comments and Data

File: app.py (Flask + Flask-SocketIO)
from flask_socketio import SocketIO, emit

socketio = SocketIO(app)

@socketio.on('post_comment')
def handle_comment(data):
    # save comment ...
    emit('new_comment', {'user': data['user'], 'text': data['text']}, broadcast=True)

def broadcast_animal_update(update):
    # called whenever an animal is added/updated/deleted
    socketio.emit('new_data', update, broadcast=True)

---

## Why These Changes Were Made

I added the switch so volunteers who need updates in real-time can get them, and others won’t be bothered. In a busy shelter, someone might mark a dog as “adopted” at any time, and volunteers need to see that right away. By sending comments and updates on a separate “new_comment” channel only when someone checks “Subscribe to Live Comments,” the dashboard becomes a true team tool. It stops volunteers from using old data but lets staff who only want to look at information keep notifications turned off so they aren’t interrupted.

---

## Reflection on Course Outcomes

Modular Design (Outcome 1): Clean separation between comment and data channels.

Real-Time Collaboration (Outcome 3): Uses WebSockets to keep all clients in sync.

User-Centered (Outcome 5): Opt-in switch prevents unwanted noise for casual viewers.