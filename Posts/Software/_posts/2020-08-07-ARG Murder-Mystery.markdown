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

I'm fortunate to have a group of friends who are not shy when it comes to role-playing. I've taken advantage of this by holding a biennial (an event occuring every two years) Murder Mystery dinner. In 2018 I took it upon myself to organize the most ambitious one yet. I would have the guests interact with each other, anonymously, through an alternate reality game (ARG).

I have to give credit to my inspiration for this: The Secret World. Yes the MMO. Prior to its launch, Funcom develop a series of ARGs from 2007 to 2012. These were incredibly detailed puzzles filled with unique imagery, lore, and riddles. They had players scouring the Internet and using all means of software to crack the codes. Each piece revealed more and more about the game itself. In the end we were rewarded with a special token which was presumably a beta key. I've yet to experience a game which has captured my imagination in the same way. 

With this ARG still in mind, I set out to make a similar experience for my guests. For me this was also an opportunity to try out some new web technologies. I had already decided on my theme for the event: 80's crime. Drugs, money, and big hair if you can imagine. Now, if the event takes place in the 80's, how do I incorporate an online ARG? Luckily there existed a beast with 64 KB of RAM and a 320x200 pixel resolution! The Commodore 64. You may have guessed from the accompanying image 🧐.

What I ended up doing was designing a secret webpage with a Commodore 64 theme, which would function as an anonymouse chatroom for my guests. I'll have to layout the entirety of the plot in another post, for this one I will cut to the goods and focus on the tech.

As a way to get my friends gears turnings, I sent them a lightly-encrypted email from my NPC, Jimmy. It went something like this:


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

You should take a minute and see if you can find the 3 clues 😉 (hover over the box below to reveal the answers!)


<div class="spoiler"><span>Dreams, 38, heart</span></div>
<br>

These clues were intended to lead my guests to the song "These Dreams" by Heart, released in 1985 — a great hit in my opinion!
From here they would need to disect the song and figure out how the "38" fits in. If one was to count the lyrics line by line down to 38, they would read "The sweetest song is silence." Entering the url <a href="www.thesweetestsongissilence.com">thesweetestsongissilence.com</a> took them to this landing page:


<img src="/post_images/thesweetestsongissilence.PNG">


I chose to work with Firebase because it offered a lightweight and ready-to-launch solution for a simple web app like this. Being a Google product it offered easy sign-in integration for my users, given that they were required to use gmail accounts. All I had to do was enable that provider and authorize my domain.
I redirect my users to a Google sign-in portal, and once verified, I pass the returned token (along with their user ID) to firebase via json. After joining, Firebase adds them to a list of authenticated users, which is then referenced by the app to determine access privileges. Here's a few code snapshots:

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

Below is what the landing page ended up looking like. Reminder that this was purposely made to look retro (Commordore 64!); but, I will admit that some of the formatting was rushed to meet my deadline. In other words its not exactly as flexible as I'd like. The left column shows the rooms that the user is part of, as well as its respective message count. The four boxes in the middle are fairly literal. The wallet in the top right corner displays the user's cash amount, which actually played a huge role throughout the game!

<img src="/post_images/commodorechat.PNG">


Let's break down each section:
### Chat Rooms

My idea with the chat rooms is that I could create two avenues for plot development throughout the game. In one manner, I would roleplay one of the NPCs and provide clues not only to the ARG, but to events for the dinner itself! The players' would have to engage with me through these rooms in order to progress in the game. I did not have enough time to set up a proper "password" system for rooms, so instead users' would need to figure out the name of the next room. I would reward the first few entrants a cash amount as motivation. The avenue of development would be facilitated with private chat rooms that could be created between users. Users could use these rooms to trade secrets, form alliances, and backstand. And they did! Here's a snap shot of one of the rooms (the final room actually):

<p style="text-align: center;"><b>Warning: Explicit (18+)</b></p>
<img src="/post_images/commodorejimmy.PNG">

Every chat room was stored as an entry inside a realtime Firebase database, which allowed for a quick and easy way to sync and retrieve data. Each room had three properties: messages, name, and people. Name was simply the name of the room, while messages and people held information such as usernames, message content, sender, date and time. The structure was like this:

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

Private rooms could be accessed by selecting that user's name from a drop down menu. I had to be a little but creative with this because if the room were to be named after the user (an exact match), anyone could access that room if they wished. This was definitely a massive flaw in my design, but I came up with a decent bandaid to fix it. All I did was appened the user's codename with 5 unique alphanumeric characters: 

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

What could have I improved on? The "Leave Room" button was decieving, it was designed to actually remove the user from the room, so that they no longer received updates for it. My users however, assumed it worked more like a back button. So they were constantly having to search for and re-join the room. I could alleviate this by designing a more obvious "Return Home" button, or perhaps using a tooltip.

### Wallet System

Another endeavour I took on was the creation of a wallet system. Like I mentioned before, money was a huge part of this game. It was to be used to negotiate, buy intel, and win prizes. I had to come up with a way for my users to make trasactions in a meaningful way.

Customer Requirements:
* Check balance
* Send money
* Receive money

Technical Requirements:
* Link user account with wallet
* Track balance
* Display balance
* Send money
    * Calculate if possible (can't send money you don't have)
* Receive money
    * Update total

The first step was to assign each registered user a "wallet" property. This was simply a field of integer value within our realtime database.
By design, I set each users' initial balance to 500. 
I would then track any changes to wallet using a store which I called "bank." I created basic actions for the store such as initialization and balance return. The less basic algorithm was my "send money" function, which needed to get both the sender and receiver IDs, subtract and add their balances respectively, and then update both wallets.<br>
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

In general, it was a fairly simple design. However, it achieved everything I needed for the event. How could I improve the system? It would have been nice to provide the user with a warning/confirmation if they were about to send cash. This way they could avoid mistakenly sending the wrong amount. I think part of this could have been handled purely using a confirm() method. And then I would go one step ahead and retrieving the user's balance, calculatin the difference in realtime.

<br>
So that's all I wanted to share regarding my little ARG project! I hope you found the read interesting, perhaps even inspiring? I'll certainly make a post later and discuss the actual plot along with the dinner's event. Stay tuned!

Cheers,

_-Rudy_