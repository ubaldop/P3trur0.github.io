---
layout: post
title: Using Github project as SBT dependency (and how to clean it up)
comments: true
spotifyTrack: spotify:track:1lsKYprMZVcKT6sLRBvuLX

---

I am developing an [utility application in Scala](https://github.com/P3trur0/packtsub) and because it needs to crawl some HTML documents, I decided to introduce an external library, SSoup, that I've forked from a Github repo some months ago.

Currently, this external dependency has not been published on a central repository, so I have no available jars to introduce as straight sbt dependency: my SSoup fork [has been hosted on Github](https://github.com/P3trur0/ssoup), so I decided to use the SBT feature to manage external Git repo projects dependencies.

## How to introduce Git project dependency

With SBT, you can refer to dependencies that are set up as GitHub projects. For those projects you don't need to have a jar file: the source code library is enough.
So, you can push your Scala libraries as source code to GitHub and then pull them into your other Scala projects by referencing them in your SBT build files.  


First, in the root of your project you need to create a subfolder named _project_.  
Within that folder, you need to create a file named _Build.scala_ with the following contents:

```scala
import sbt._

object PacktSubBuild extends Build {
  
  lazy val root = Project("root", file(".")) dependsOn(ssoup)
  lazy val ssoup = RootProject(uri("https://github.com/P3trur0/ssoup.git")) 
  
}
```

This code is self-explaining: it states that my **"root"** project `dependsOn` the **ssoup** value.  
This **ssoup** value refers to the Github repository through the **uri** specified.  

Then, you are ready to run your SBT compile tasks by invoking the command `sbt run`.  
At the end of the SBT execution you should see that the external Github dependency has been properly imported in your environment.  
In detail, if you browse the **~/.sbt/0.13/staging/** folder you can see some subdirectory as the following: **fca17166eb70c70bf98e**.  
It contains the source code of the external Github imported dependency.

## How to clean it up

So far it looks awesome. However, once imported, the folder contained in  
**~/.sbt/0.13/staging/** is not refreshed anymore.  
This means that if you perform any change on the external dependency, push it to Github and run your SBT tasks again, then SBT does not recompile the external Github dependency again when compiling your project.  
Basically, you need to manually clean up the SBT staging directory each time you updates your external Github dependencies.  
This is a little annoying (and someone has already [opened a bug](https://github.com/sbt/sbt/issues/1284) to the SBT dev team).  

So, workaround time here!  

In your _build.sbt_ file, you can write a little SBT task that helps to perform this cleaning using SBT itself.  
Here it is the little script I wrote to clean up the staging directory.

```scala
def stagingClean = Command.command("staging-clean"){currentState =>
	val homeDir = sys.env("HOME")
	val k = ("rm -rf "+ homeDir + "/.sbt/0.13/staging/").!
	currentState
}

commands ++= Seq(stagingClean)
```

This script defines a simple SBT task, named `staging-clean`, that removes all the folders currently contained in **~/.sbt/0.13/staging/**.

If you run this task before the SBT compilation task you force the tool to download the proper external Git dependencies again. Then all the updates, if any, will be introduced in your project.

I hope you find it useful as I did.

Do you have any suggestions/improvement about this approach?
