---
title: SBT Plugin development (testing)
layout: post
---

This is the first in a series of posts that will describe my voyage in accepting appropriate `SBT` way of thinking.

Assuming that you are comfortable with scala, sbt, after that have a good idea why you would need a plugin.


#### Its easy to write a plugin!
Add to your build.sbt, or settings of a subproject.

```scala
sbtPlugin := true
```

that's it, good job there.


#### Now, test stuff

Add to the plugins (`project/plugins.sbt`)

```scala
libraryDependencies <+= 
    sbtVersion("org.scala-sbt" % "scripted-plugin" % _)
```

Add to your project settings:

```scala
ScriptedPlugin.scriptedLaunchOpts := { 
  ScriptedPlugin.scriptedLaunchOpts.value ++
  Seq("-Xmx1024M", 
      "-XX:MaxPermSize=256M", 
      "-Dplugin.version=" + version.value
  )
}
```

Those look like some variables that scripted will put into a JVM where it will test your plugin. Right?

#### An Actual test
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

1. `plugin.sbt` containing

    ```scala
    addSbtPlugin("your.groupId" % "your-app-name" % 
        System.getProperty("plugin.version"))
    ```

    This explains that variable you have placed into the `scriptedLaunchOpts`. You wouldn't want to fix every test every time you bump a version.

2. `build.sbt` is where you might want to define some keys which define a setting you are testing. For now, we'll just define a `task`

    ```scala
    TaskKey[Unit]("onePlusOneIsTwo") := {
      assert(1 + 1 == 2)
    }
    ```
s

3. `test` is a file that will be called on each scripted run. Here you can place anything that can be written to the sbt console of your test project. Lets just call to perform the assertion above.  

        > onePlusOneIsTwo


There, testing is easy, we acutely wrote a task along the way, fun times!

Call all tests with 

    > scripted

To perform a single test call 

    > scripted test_group/test_n


Scripted will now `publishLocal` your plugin and resolve it in a project it copied to `/tmp/sbt-<some randoms>` and then run a task you called in `test` file. 
If your task returns some values you could call it from the task you defined in the `build.sbt` and `assert` the returned value with an expected result.
If your project depends on a library you are developing as a subproject, you have to publish it too. To do so just add it to the `publishLocal` task in your plugin project.

```scala
publishLocal <<= publishLocal dependsOn( 
    publishLocal in yourLibraryProjectRef )
```


Similarly for the tests, test resources in the library projects, only this is a path, just add them to the classpath with something like

```scala
unmanagedResourceDirectories in Test <++= 
    unmanagedResourceDirectories in Test in <subproject>
```

Then if you are using some test resources or mocks you need to load them with your plugin to the test project. So your test `plugin.sbt` file now looks like this:

```scala
addSbtPlugin("your.groupId" % "your-app-name" % 
    System.getProperty("plugin.version"))
        classifier "tests" classifier "")
```

####Links:

[Eugine Yokota on testing](http://eed3si9n.com/testing-sbt-plugins)

