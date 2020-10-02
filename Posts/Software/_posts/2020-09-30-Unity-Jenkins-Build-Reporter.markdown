---
layout: post
title: "Unity CI Build Reporter"
date: 2020-09-30 10:12:00 -0000
categories: software
image: bug_reporter_app.PNG
summary: "I combine Unity, Jenkns, ExpressJS, VueJS, and MySQL to create a fullstack build reporting system."
tags: NodeJS ExpressJS VueJS MySQL Unity Jenkins
---

<!-- The Modal -->
<div id="myModal" class="modal">
  <span class="close">&times;</span>
  <img class="modal-content" id="img01">
  <div id="caption"></div>
</div>

### Project Highlights:
* [Motivation](#BuildReport-Motivation)
* [Setting up the Jenkins Build](#BuildReport-Jenkins)
* [Database Connection](#BuildReport-Database)
* [Backend](#BuildReport-Backend)
* [Frontend](#BuildReport-Frontend)

### <a id="BuildReport-Motivation"></a> Motivation
I recently posted an article about a [Game Crash Report](https://rudylang.github.io/posts/software/2020/09/14/Game-Crash-Reporter.html) which got me thinking a bit more about game development tools, especially from the perspective of continuous integration. Often team members are collaborating on a project, and have set rules in regards to how content can be merged. Having a working build would certainly be one of them. Jenkins is a great tool for running builds behind the scenes and automating various tasks. However, what tends to happen with Jenkins is that builds can be obfuscated by volume or information. My solution is to design a web interface that consolidates the information that matters.

Automation can take on many different forms, and have different meanings depending on what you're working on. I'll share some high level thoughts on how I view the process in regards to Unity development:

* Unity builds can be run directly from the editor or from the command line
* Unity build logs are generated from this this process, and supply the developer with useful information
* There can exist a database that stores these reports
* A web application can read from this database, and share the results with any who has access

There are components of this process that can exist as scripts. And these scripts can be run automatically using Jenkins:

* A Unity project can exist on github which developers will interact with directly
* Unity command line arguments can be run as a batch script to build the project
* Unity command line arguments can produce a build log
* A python script can parse through this build log and grad the information of interest
* The same python script can make MySQL queries and communicate with our database
* A backend API can be running and also make calls to the database to get this data
* Lastly, the front end makes a fetch call to recieve this data, and reorganize it into a readable format

### <a id="BuildReport-Jenkins"></a> Setting up the Jenkins Build
I opted to keep things simple and run the Jenkins server locally from my Windows desktop. Ideally, you would have a separate machine running the server. Jenkins has many plugins available to ease the integration of your tools. Luckily enough, someone has built a plugin called [Unity3d](https://plugins.jenkins.io/unity3d-plugin/) which will do a lot of the heavy lifting for us! With the plugin installed, your only real job is to specify where you have the Editor installed, like so:

<img class="myImg" src="/post_images/build_unity_plugin.PNG">

Once that's been saved, a freestyle project can be created. Under "Source Code Management" I specified the repo where my Unity project lives, the branch it lives on, and my SSH credentials:

<img class="myImg" src="/post_images/build_git.PNG">

Within the Build Environment, I opted to abort the build if it hangs for 30 minutes. I also chose to simply use the build number as the build name using the plugin "Build Name and Description Setter":

<img class="myImg" src="/post_images/build_environment.PNG">

This next part is the meat and potatoes. Because I've installed the Unity3d plugin, I can select it as a build option. All it requires is a name and command line arguments:

<img class="myImg" src="/post_images/build_unity_build.PNG">

Here are the commands broken down:


| Command       | Description           |
| ------------- |:-------------:|
| `-nographics`      | Will not initialize the graphics device (no window). |
| `-batchmode`      | Run Unity in batchmode, which means no pop-ups that require human interaction. Allows automation without interuption.      |
| `-quit` | Quit the Unity Editor once the commands are done.      |
| `-executeMethod` | Execute a static method that exists within the Unity project. This will allow us to run the build for the Windows64 target.      |
| `-logFile` | Specify where the log files are written onces the build is done.     |
| `JenkinsBuild.BuildWindows64` | This is the specific build function that we want to run.      |

All together:

<div class="article-code">
<pre>
-nographics -batchmode -quit 
-executeMethod JenkinsBuild.BuildWindows64 "${JOB_NAME}"
"C:\Jenkins\${JOB_NAME}\Builds\${BUILD_NUMBER}\output" 
-logFile "C:\Jenkins\${JOB_NAME}\Builds\${BUILD_NUMBER}\output\Logs\UnityEditor.log"
</pre>
</div>

Most of what I'm doing comes from this fantastic individual here: [GameFeelings](https://www.youtube.com/watch?v=WdIG0af7S0g). I've used the build script that they've written [here](https://gamefeelings.com/2020/02/18/jenkins-build-with-unity3d/). I really appreciated their unsolicited guidance!

The last portion is my Python script, which servers to parse the generated logfile and query the database with the results. Because I'm running the Jenkins server on Windows, I'll be using a batch script to invoke Python. Now I've kept things simple by pushing my script with the Unity project so that it's in the same workspace. I could also let the script live locally on the Jenkins server. It looks like this:

<img class="myImg" src="/post_images/build_python_batch.PNG">
<div class="article-code">
<pre>
C:\Users\RudyL\Anaconda3\python.exe report_parser.py 
"C:\Jenkins\%JOB_NAME%\Builds\%BUILD_NUMBER%\output\Logs\UnityEditor.log"
</pre>
</div>


### <a id="BuildReport-Database"></a> Database Connection

For the database connection all I wanted was a python script that would feed data to the MySQL database. The first thing I needed to determine was what information I wanted to store. After reading through the generate build log, I found a few convenient options:

* Build Number
* Build Type (Release or Development)
* Build Size
* Build Time
* Build Status (Success or Fail)

These items were sufficient given the small scope of the project. Now know this information, I set up a new table in my builds database:

<img class="myImg" src="/post_images/build_sql_table.PNG">
