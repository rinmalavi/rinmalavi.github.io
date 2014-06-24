---
title: SBT Plugin development (from a point of view)
layout: post
published: false
---

Assuming that you are good with scala, sbt, after that have a good idea why you would eve want to do a plugin.

### So you want to write a plugin?!

    sbtPlugin := true

that's it, good job there.

###Now, test stuff

Add to the plugins (`project/plugins.sbt`)

    libraryDependencies <+= sbtVersion("org.scala-sbt" % "scripted-plugin" % _)

And to your project settings:

    ScriptedPlugin.scriptedLaunchOpts := { 
      ScriptedPlugin.scriptedLaunchOpts.value ++
      Seq("-Xmx1024M", "-XX:MaxPermSize=256M", "-Dplugin.version=" + version.value)
    }

Those look like some variables scripted will put into JVM where it is testing your plugin. Right?

#####An Actual test
looks like this. In `src/sbt-test` make a folder within which you will add folders (don't ask just do...)

    .../sbt-test
        └── group_of_tests_I_guess
            ├── test_1
            │   ├── build.sbt
            │   ├── test
            │   └── project
            │       ├── build.properties
            │       └── plugins.sbt
            └── test_2 ...

Only 3 files need to be explained here:

1. `plugin.sbt` containing `addSbtPlugin("your.groupId" % "your-app-name" % System.getProperty("plugin.version"))`
Which explains that variable you have put in to `scriptedLaunchOpts`. You wouldn't want to fix every test every time you bump a version, or worse...

2. `build.sbt` is where you might want to define some keys which define a setting you are testing. For now, will just define a `task`


    TaskKey[Unit]("onePlusOneIsTwo") := {
      assert(1 + 1 == 2)
    }

3. `test` `>onePlusOneIsTwo`

There testing is easy, we acutely wrote a task along the way, fun times!

Call all tests with `scripted`.
To perform a single test call `scripted test_group/test_n`.