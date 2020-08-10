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


These clues were intended to lead my guests to the song "These Dreams" by Heart, released in 1985 — a great hit in my opinion!
From here they would need to disect the song and figure out how the "38" fits in. If one was to count the lyrics line by line down to 38, they would read "The sweetest song is silence." Entering the url <a href="www.thesweetestsongissilence.com">thesweetestsongissilence.com</a> took them to this landing page:


<img src="/post_images/thesweetestsongissilence.PNG">


I chose to work with Firebase because it offered a lightweight and ready-to-launch solution for a simple web app like this. Being a Google product it offered easy sign-in integration for my users, given that they were required to use gmail accounts. All I had to do was enable that provider and authorize my domain.
I redirect my users to a Google sign-in portal, and once verified, I pass the returned token (along with their user ID) to firebase via json. After joining, Firebase adds them to a list of authenticated users, which is then referenced by the app to determine access privileges. 