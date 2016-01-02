# ENSIME SBT

This [sbt](http://github.com/sbt/sbt) plugin generates a `.ensime` file and provides various convenience commands for interacting with [ENSIME](http://github.com/ensime/ensime-server).

## Install

Add these lines to `~/.sbt/0.13/plugins/plugins.sbt` as opposed to `project/plugins.sbt` (the decision to use ENSIME is per-user, rather than per-project):

```scala
addSbtPlugin("org.ensime" % "ensime-sbt" % "0.3.0")
```

**Check that again**, if you incorrectly used `~/.sbt/0.13/plugins.sbt` you'll get an sbt resolution error, it really has to be in the `plugins` folder.

Alternatively, copy the `EnsimePlugin.scala` into your `project` directory and make sure you have `scalariform` and `scalap` on your project definition's classpath. This approach works well in environments that do not have access to maven central.

## Commands

* `gen-ensime` --- Generate a `.ensime` for the project (takes space-separated parameters to restrict to subprojects).
* `gen-ensime-project` --- Generate a `project/.ensime` for the project definition.
* `debugging` --- Add debugging flags to all forked JVM processes.
* `debugging-off` --- Remove debugging flags from all forked JVM processes.

Note that downloading and resolving the sources and javadocs can take some time on first use.

(Copied from [EnsimePlugin.scala](https://github.com/ensime/ensime-sbt/blob/master/src/main/scala/EnsimePlugin.scala#L59))

### Debugging Example

If you do not `fork` your main methods and tests from `sbt` you may be able to attach a remote debugger to the entire session by starting like `sbt -jvm-debug 1337` (check the docs of your `sbt` script, we strongly recommend using [paulp's sbt-extras](https://github.com/paulp/sbt-extras)). However, the vast majority of projects enabled forking and that is when `ensime-sbt`'s `debugging` is useful. 

This sample session shows how easy it is to remotely debug a forked test:

```
crossbuild ~/Projects/ensime-server sbt
> debugging
[warn] Enabling debugging for all forked processes
[info] Only one JVM can use the port and it will await a connection before proceeding.
> jerk/test-only *JerkFormatsSpec
Listening for transport dt_socket at address: 5005
```

at which point, the test will hang until you connect a remote debugger to port 5005. When you are finished debugging, cancel the test or let it run to completion, and then type `debugging-off`.

Note: If you'd like to debug using ensime-emacs, first set your breakpoints, then use ensime-db-attach to connect.

## Customise

Customising [EnsimeKeys](https://github.com/ensime/ensime-sbt/blob/master/src/main/scala/EnsimePlugin.scala#L21) is done via the usual sbt mechanism, e.g. insert the following into `~/.sbt/0.13/ensime.sbt`

```scala
import org.ensime.Imports.EnsimeKeys

EnsimeKeys.debuggingPort := 1337
```

## Troubleshooting

Always check the [tickets flagged as FAQ](https://github.com/ensime/ensime-sbt/issues?q=label%3AFAQ) before reporting a new issue.

Tickets are extremely hard to reproduce unless you create a minimal example project and share with us the steps to reproduce the problem.

Bug reports in the form of a pull request into the `src/sbt-test/ensime-sbt` directory are well received, which is our suite of integration tests using the [sbt scripted](http://eed3si9n.com/testing-sbt-plugins) framework (its very simple to use). Pull requests with tests and fixes are living the dream.

You can follow snapshot releases by using the following instead of the stable release

```scala
// or clone this repo and type `sbt publishLocal`
resolvers += Resolver.sonatypeRepo("snapshots")

// update to the latest development version, see project/EnsimeSbtBuild.scala
addSbtPlugin("org.ensime" % "ensime-sbt" % "0.3.1-SNAPSHOT")
```


### Cancel Debugging Processes

By default, `sbt` will not cancel a running subprocess if you type `C-c`. You are recommended to add the following to your `~/.sbt/0.13/global.sbt` so that it will do so:

```scala
cancelable in Global := true
```

Emacs users should recall that in order to send a control sequence to the `sbt-mode` subprocess, first run `sbt-clear` (bound to `C-c C-v` by default) then `C-c`. i.e. to send `C-c` to `sbt`, type `C-c C-v C-c C-c` (you probably want to bind this to something easier).

### Formatting

If you use [sbt-scalariform](https://github.com/sbt/sbt-scalariform), and wish to use the same settings in ENSIME, you must set in your build `settings`:

```scala
EnsimeKeys.scalariform := ScalariformKeys.preferences.value
```

e.g. for `Build.scala` style projects

```scala
import org.ensime.Imports.EnsimeKeys

override val settings = super.settings ++ Seq(
  ScalariformKeys.preferences := <... your scalariform settings here ...> ,
  EnsimeKeys.scalariform := ScalariformKeys.preferences.value
)
```

This is not automatic in order to workaround https://github.com/daniel-trinh/sbt-scalariform/issues/24

### SBT Version

Your `project/build.properties` needs to use a version newer than 0.13.5 of sbt due to a [breaking AutoPlugin change](https://github.com/ensime/ensime-server/issues/672), e.g.

```
sbt.version=0.13.9
```
