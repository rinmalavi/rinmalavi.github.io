---
title: SBT Chain Task and LastModified
layout: post
---

A short post on how you might add your task to preform in chain with other tasks. 

[plugins doc](http://www.scala-sbt.org/0.13/docs/Plugins.html)

Previously there was a [post on testing](/2014/07/04/sbt-plugin-testing-basics/).
Being able to automatically test code you write, especially if you are still learning,
will definitely make up for time spent writing tests.

#### Keys And Basics

Key can be defined as one of 3 different types:

1. `SettingsKey` on load

2. `TaskKey` on call

3. `InputKey` on call with arguments


Shorthand way to define them
```scala
TaskKey[Unit]("callTask") := {
  println("doing something here")
}
```

Define a key separately from the task initialization.

```scala
lazy val someTask = taskKey[Unit]("task description")

someTask <<= taskDef

private def taskDef: Def.Initialize[Task[Unit]] = Def.task {
  println("doing something here")
}
```

Tasks return values

```scala
TaskKey[Unit]("callTask") := {
  2
}
```
    
## Chain tasks

Values are resolved as soon as they enter the scope.
To make a task depend on another task use `Def.taskDyn`

```scala
def superTask = Def.taskDyn{
  if (shouldIcontinue.value) {
    Def.task {
      theContinuation.value
    }
  } else Def.task{}
}
```

### Make a db and upgrade ahead of the tests

Find a file defining your model. 
Should be placed in watch so the modification could trigger the task recall.
 
```scala
val curr = 
watchSources
  .value
  .seq
  .find(
    _.getName
    .contains("model.sql"))
    .headOption
    .foreach{
  f => if (System.currentTimeMillis() - f.lastModified() < 9999)
            createDB()
}
```
