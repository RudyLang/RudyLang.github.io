---
layout: post
title: "ARG - Murder Mystery"
date: 2020-08-07 13:57:00 -0000
categories: software
image: mmcommodore.png
summary: "A Murder Mystery Dinner goes virtual! I put my guests through an Alternate Reality Game (ARG) before the big event."
tags: React Redux Firebase
featured: true
---

<img src="/post_images/mmcommodore.png">

### Project Highlights:
* [Motivation](#ARG-Motivation)
* [Firebase](#ARG-Firebase)
* [Chat Rooms](#ARG-Chat)
* [Wallet System](#ARG-Wallet)

### <a id="ARG-Motivation"></a> Motivation
My friends love to roleplay, so I hold a Murder Mystery dinner event every two years. In 2018, I organized the event with an alternate reality game (ARG) component. I was inspired by Funcom's MMO, The Secret World, which used ARG puzzles to promote the game before it's release. The games were detailed puzzles full of unique imagery and lore. The end reward was a special token, a beta key. No game since has captured my imagination quite the same way. 

I set out to make a similar experience for my friends. The event's theme was 80's crime. Think drugs, money, and big hair. I would also use the pinnacle of 80's technology, the Commodore 64, as the theme for the ARG. I sent my friends a message from my character, the crime boss's assistant Jimmy. In the message, they would find the password to log onto a secret Commodore 64-themed web page.

Here was the opening message: 



<p style="text-align: center;"><b>Warning: Explicit (18+)</b></p>
<div class="break">&nbsp;</div>
<p class="quote">
‘Sup,<br><br>

Ya might be wonderin' why I’ve contacted ya. Ya see, I know the kind of crap ya’ve been up ta, but guess what, I don’t mind. In fact, I admire it. I also know that ya could use some cash. Well I have cash, a f*****’ lot of it. And I don’t mind givin’ it away if I get what I need. I hope ya know where I’m goin’ with this.<br><br>   

Ya see the Boss is a busy man, he’s always travelin’ and makin’ deals. That’s why he brought me on. I’m good at findin’ people and findin’ things. He’s asked me to put togetha a crack team to get some shit done, ya know, under the table. To be frank this isn’t an opportunity ya should turn down.<br><br>

The Boss and I have put together somethin’ revolutionary. Ain’t nobody’s done this. We’ve built a communication tool completely on the web. You f*****’ heard me right, no phones, the whole thing is on the computa. I gotta give most of credit to the Boss though, the guy’s a genius, I simply pitched the idea to em.<br><br>

Ya probably wonderin’ who the f*** we is, well I can tell ya that ya’ll get to know me quite well in the next couple weeks if you decide to join. The Boss, well he’s a more complex characta. He had some big Dreams that didn’t quite pan out. Came to a new realizatin’ when he got to 38. I’d say things are lookin’ up for em now. You know, the 80s does have alota heart.<br><br>

But anyway, enough of my ramblin’. If ya think ya got what it takes, ya’ll be able to find this tool of ours no problem. The first 7 of ya will get a nice bonus, how bout that!<br><br>

Later days and better lays,<br><br>
-Jimmy<br><br>


*META ON*<br><br>
Find the webapp. It is comprised of 5 words and has the .com domain.
Be logged into your anonymous gmail account while you do this. When you log into the web app you’ll be prompted with all the available google accounts associated with you. Select the one with your codename of course.
Once in, join the room “Jimmy” by entering it into the “Join a Room” box. Ignore the other features at this time. You can navigate between the landing page and the chat rooms by clicking “Commodore” in the top left.<br>
</p>

There are three clues in the message. You can hover over the box below to reveal them!

<div class="spoiler"><span>Dreams, 38, heart</span></div>
<br>

The password was the 38th line from the song "These Dreams: by Heart, released in 1985. The URL for the website was the lyric "[thesweetestsongissilence.com](thesweetestsongissilence.com)". This was the landing page: 


<img src="/post_images/thesweetestsongissilence.PNG">


### <a id="ARG-Firebase"></a> Firebase

I chose to work with Firebase because it offered a lightweight and ready-to-launch solution for a simple web app like this. As a Google product, it offered easy sign-in integration for my users, who used Gmail accounts. All I had to do was enable that provider and authorize my domain. I redirect my users to a Google sign-in portal, and once verified, I pass the returned token (along with their user ID) to Firebase via JSON. After joining, Firebase adds them to a list of authenticated users, which the app references to determine access privileges. Here are a few code snapshots:

<div class="article-code">
<pre>
// Import Firebase module and initialize Google Authentication

import * as firebase from 'firebase';

const config = {
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: process.env.FIREBASE_AUTH_DOMAIN,
  databaseURL: process.env.FIREBASE_DATABASE_URL,
  projectId: process.env.FIREBASE_PROJECT_ID,
  storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID
};

firebase.initializeApp(config);

const database = firebase.database();
const googleAuthProvider = new firebase.auth.GoogleAuthProvider();

export { firebase, googleAuthProvider, database as default };
</pre>
</div>

<div class="article-code">
<pre>
// Invoke login function

export const startLogin = () => {
  return () => {

    return firebase.auth().signInWithRedirect(provider);

    firebase.auth().getRedirectResult().then(function(result) {

      // Return Google Access Token
      var token = result.credential.accessToken;

      // The signed-in user info
      var user = result.user;
      const name = user.displayName ? user.displayName : user.email;

      database.ref(`users/${user.uid}`).once((snapshot) => {
        if(!snapshot.val()) {
          database.ref(`users/${user.uid}`).set({
            name,
            uid: user.uid,
            email: user.email,
            rooms: [],
            token
          });
        }
      });

      ...
</pre>
</div>


Throughout the rest of the app, I employed an observer to verify if the user is logged in:

<div class="article-code">
<pre>
// Verify user exists
// If true, store user ID and display name
// Else, logout, clear states, and redirect to login page

firebase.auth().onAuthStateChanged((user) => {
  if (user) {
    const name = user.displayName ? user.displayName : user.email;    
    store.dispatch(login(user.uid, name));
    store.dispatch(setStartState());    
    renderApp();
    if (history.location.pathname === '/') {
      history.push('/join');
    }
  } else {
    store.dispatch(logout());
    store.dispatch(clearState);
    renderApp();
    history.push('/');
  }
});
</pre>
</div>

After log-in, my users were greeted with this retro Commodore 64-like page. I rushed the formatting a bit to make it in time for the event, but I'm pleased with the results. The left column shows the chat rooms the user has joined, and the top right corner is the user's wallet. The four boxes in the middle are literal. 


<img src="/post_images/commodorechat.PNG">

<br>
Let's break down each section:

### <a id="ARG-Chat"></a> Chat Rooms

The chat room provided the first avenue for plot development. The second opportunity would be the in-person dinner held a few weeks later. I would roleplay Jimmy in both contexts. The players could engage with me, or each other, in both as well. 
In order to access the chat rooms, the users had to figure out the name of the room. The first few users to get it right got a cash award in-game. Once in the room, users could trade secrets, form alliances and backstab each other, and they did! Here's a snapshot of the final room: 


<p style="text-align: center;"><b>Warning: Explicit (18+)</b></p>
<img src="/post_images/commodorejimmy.PNG">

Every chat room was stored as an entry inside a real-time Firebase database, which allowed for a quick and easy way to sync and retrieve data. Each room had three properties: messages, name, and people. Name was simply the name of the room, while messages and people held information such as usernames, message content, sender, date and time. This was the structure:

<div class="article-code">
<pre>
Project
    roomname
        RoomA
            messages
                messageID
                    createdAt: "2018-10-14T16:35:23-04:00"
                    sender
                        displayName: "Person"
                    status: true
                    text: "This is a new message."
            name: "RoomA"
            people
                userID
                    id: "XXXX"
                    lastRead ""2018-11-03T14:27:11-04:00"
                    name: "User Name"
                    undread: 0
</pre>
</div>


I could access the user's private room by selecting their name from a drop-down menu. Although, it couldn't be an exact match for their name or anyone could access the room. This was a flaw in my design, but I added five unique alphanumeric characters to each name to solve the problem. 

<div class="article-code">
<pre>
// Initialize data to invoke private room creation

onCreatePrivateRoom = (e) => {

    ...

    let private_room_name = other_user_name + this.makeid();

    ...
}

// Helper function to create random ID
makeid() {
    var random_key = "";
    var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

    for (var i = 0; i < 5; i++)
      random_key += possible.charAt(Math.floor(Math.random() * possible.length));

    return random_key;
  }
</pre>
</div>

I also could have improved on the "leave room" button. The users assumed it worked like a back button, but it was actually designed to remove them from the room entirely. So, they had to search for and rejoin the room when they used it. I could have designed a "return home button" or used a tooltip to fix this.

### <a id="ARG-Wallet"></a> Wallet System

Money was a huge part of the game. Users could buy intel, negotiate and win prizes with it. So, they had to be able to make meaningful transactions. 

Customer Requirements:
* Check balance
* Send money
* Receive money

Technical Requirements:
* Link user account with wallet
* Track balance
* Display balance
* Send money
    * Calculate if it's possible to send money (so users can't send money they don't have)
* Receive money
    * Update money

First, I assigned each user a "wallet" property. This wallet was simply a field of integer value within the real-time database. By design, I set each users' initial balance to 500. I would then track any changes to wallet using a store that I called "bank." I created basic actions for the store, such as initialization and balance return. I also added a "send money" function, which needed to get both the sender and receiver IDs, subtract and add their balances respectively, and then update both wallets.

I set it up like this:

<div class="article-code">
<pre>
// Substract cash value from sender waller, add that amount to reciever wallet.
// If sender wallet does not have appropriate funds, bypass the database update.

export const startSendMoney = (cash, receiver_user_id, sender_user_id) => {
    return (dispatch, getState) => {

        let noFunds = false;

        return database.ref(`users/${sender_user_id}/wallet`).once('value', (snapshot) => {
            const value = snapshot.val();
            if (value === null) {
                console.log("Where's my wallet!");
            } else {
                const update_balance = value.balance - parseInt(cash);
                if (update_balance >= 0) {
                    database.ref(`users/${sender_user_id}/wallet`).set({ balance: update_balance });
                    dispatch(getWalletBalance(update_balance));
                } else {
                    // If the updated_balance is <= 0,
                    // then the sender does not have the appropriate funds; set flag
                    noFunds = true;
                }
            }
            return database.ref(`users/${receiver_user_id}/wallet`).once('value', (snapshot) => {
                const value = snapshot.val();
                if (value === null) {
                    console.log("Where's my wallet!");
                } else {
                    const update_balance = value.balance + parseInt(cash);

                    // Only update the receivers wallet property if there were enough funds available
                    if (!noFunds) {
                        database.ref(`users/${receiver_user_id}/wallet`).set({ balance: update_balance });
                    }
                }
            });
        });
    }
}
</pre>
</div>

This reasonably simple design achieved everything I needed for the event. Although, I did see a few ways I could improve the project. It would have been nice to provide the user with a warning/confirmation if they were about to send cash. This way, they could avoid mistakenly sending the wrong amount. I could have handled part of this using a confirm() method. And then, I would go one step ahead and retrieve the user's balance, calculating the difference in real-time.

<br>
The ARG portion of the event was a success. All 14 people participated, chatted in several rooms and made several transactions. In a the later post, I'll discuss the actual plot and the in-person event! 

Cheers,

_-Rudy_