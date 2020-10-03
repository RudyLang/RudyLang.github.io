---
layout: post
title: "Unity CI Build Reporter"
date: 2020-09-30 10:12:00 -0000
categories: software
image: build_reporter_main.PNG
summary: "I combine Unity, Jenkns, ExpressJS, VueJS, and MySQL to create a fullstack build reporting system."
tags: NodeJS ExpressJS VueJS MySQL Unity Jenkins
featured: true
---

<img class="myImg" src="/post_images/build_reporter_main.PNG">

<!-- The Modal -->
<div id="myModal" class="modal">
  <span class="close">&times;</span>
  <img class="modal-content" id="img01">
  <div id="caption"></div>
</div>

### Project Highlights:
* [Motivation](#BuildReport-Motivation)
* [Setting up the Jenkins Build](#BuildReport-Jenkins)
* [Database Setup](#BuildReport-Database)
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


### <a id="BuildReport-Database"></a> Database Setup

For the database connection all I wanted was a python script that would feed data to the MySQL database. The first thing I needed to determine was what information I wanted to store. After reading through the generate build log, I found a few convenient options:

* Build Number
* Build Type (Release or Development)
* Build Size
* Build Time
* Build Status (Success or Fail)

These items were sufficient given the small scope of the project. Now knowing this information, I set up a new table in my builds database:

<img class="myImg" src="/post_images/build_sql_table.PNG">

These field names are what I'll be using to query the database. Python conveniently has a mysql connector that you can import and configure. Here's a condensed snippet of the general setup:

<div class="article-code">
<pre>
import mysql.connector

# Setup MySQL connector
mydb = mysql.connector.connect(
    host="localhost",
    username="Rudy",
    password="mypassword",
    port="3306",
    database="builds"
)

mycursor = mydb.cursor()

sql = "INSERT INTO unity (
  build_number, 
  build_type, 
  build_size, 
  build_time, 
  build_status, 
  build_link
) 
VALUES (%s, %s, %s, %s, %s, %s)"
val = (
  build_number, 
  build_type, 
  build_size, 
  build_time, 
  build_status, 
  build_link
)
mycursor.execute(sql, val)

mydb.commit()
</pre>
</div>

The values for these fields can be parsed in various ways. I went for the line by line approach and searched for unique substrings.
Here's an example:
<div class="article-code">
<pre>
for line in Lines:
      if "Complete build size" in line:
        build_size = line.split("Complete build size    ",1)[1]
        build_size.strip()
</pre>
</div>

The split() function is very handy as it will just grab the rest of string following a specified substring. And it's a good idea to use strip() to remove any leading or trailing white spaces. For cases where the information is nested between two substrings, I used search():

<div class="article-code">
<pre>
if "Build Type" in line:
  build_type = re.search("Build Type '(.*)'", line)
  build_type = build_type.group(1)
  build_type.strip()
</pre>
</div>

Normally search() is used to find a specific regular expression; however, all I'm telling it to do is grab any pattern (hence the greedy wild card '.*') between two substrings. The group() method just allows me to access those captured substrings. 

Admittedly that's all there is to it. A real minimal-viable product!

### <a id="BuildReport-Backend"></a> Backend

I made things easy for myself and recycled the backend from my [Game Crash Reporter](http://localhost:4000/posts/software/2020/09/14/Game-Crash-Reporter.html).
So essentially Express with the MySQL node module. I set up my model, controller, and route like so:

<div class="article-code">
<pre>
// model.js
var mysql = require('./db.js');

var Build = function(build) {
    this.build_number = build.build_number;
}

Build.getAllBuilds = function (result) {
    mysql.query("SELECT * FROM unity", function (err, res) {
        if (err) {
            result(null, err);
        }
        else {
            console.log('builds: ', res);
            result(null, res);
        }
    });
};

module.exports = Build;
</pre>
</div>

<div class="article-code">
<pre>
// controller.js
var Build = require("../model/model")

exports.list_all_builds = function (req, res) {
    Build.getAllBuilds(function(err, build) {
        if (err) {
            res.send(err);
            console.log('res', build);
        }
        res.send(build);
    });
};
</pre>
</div>

<div class="article-code">
<pre>
// route.js
module.exports = function(app) {
    var builds = require('../controller/controller');

    app.route('/builds')
        .get(builds.list_all_builds);
};
</pre>
</div>

### <a id="BuildReport-Frontend"></a> Frontend

And once again I turn to my good friends Vue and Vuetify to solve my frontend problems. Vuetify has these really fantastic data table components that were perfect for this project. They even come with a "filterable" property that allows for native search, right in the component! No funny business.
What I had to do was populate the headers with values that matched those within database:

<div class="article-code">
<pre>
  data() {
    return {
      builds: [],
      search: "",
      headers: [
        { text: "Number", align: "center", sortable: true, value: "build_number" },
        { text: "Type", sortable: true, value: "build_type" },
        { text: "Size", sortable: true, value: "build_size" },
        { text: "Time", sortable: true, value: "build_time" },
        { text: "Status", sortable: true, value: "build_status" },
        { text: "Timestamp", sortable: true, value: "timestamp" },
        { text: "Link", sortable: true, value: "build_link" }
      ],
    };
  },
</pre>
</div>

This way when I get my response from the backend, I can feed the data directly into that 'builds' array. And that array is then passed to the ":items" property for the data table!

<div class="article-code">
<pre>
  async created() {
    try {
      let res = await this.$http({
        url: 'http://localhost:3000/builds',
        method: 'get',
        timeout: 8000,
        headers: {
          'Content-Type': 'application/json',
        }
      });
      if(res.status == 200){
          console.log(res.status);
      }
      this.builds = res.data;    
      return res.data;
    }
    catch (err) {
      console.error(err);
    }
  },
</pre>
</div>

<div class="article-code">
<pre>
// In the template
 <v-data-table 
      :headers="headers"
      :items="builds"
      :search="search"
      :items-per-page="10">

    ...
  </v-data-table>
</pre>
</div>

Let's see the whole thing in action!

<img class="myImg" src="/post_images/build_reporter.gif">