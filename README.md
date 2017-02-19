# Ceylon Gradle Plugin

[![Build Status](https://travis-ci.org/renatoathaydes/ceylon-gradle-plugin.svg?branch=master)](https://travis-ci.org/renatoathaydes/ceylon-gradle-plugin)
[ ![Download](https://api.bintray.com/packages/renatoathaydes/maven/com.athaydes.ceylon/images/download.svg) ](https://bintray.com/renatoathaydes/maven/com.athaydes.ceylon/_latestVersion)

This is a Gradle Plugin to make it easy to handle Ceylon projects with Gradle.

Because Ceylon itself manages Ceylon dependencies, this plugin focuses on
managing legacy Java dependencies so that you can get transitive dependencies
resolved by Gradle.

This plugin can be a real lifesaver while the Ceylon ecosystem is still developing and you need 
to rely on Java (and Scala, Groovy, Kotlin)'s libraries!

Check out my [blog post](https://sites.google.com/a/athaydes.com/renato-athaydes/posts/theceylongradleplugin)
about this plugin to understand more about the motivations behind this project.

## Using this plugin

To use this plugin, simply add a Gradle build file to the root of your project
and apply this plugin as shown below:

* Gradle 2.1+ and  3.0+

```groovy
plugins {
    id 'com.athaydes.ceylon' version '1.3.0'
}
```

* Old style Gradle plugin declaration

*build.gradle*
```groovy
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath "com.athaydes.gradle.ceylon:ceylon-gradle-plugin:1.3.0"
    }
}

apply plugin: 'com.athaydes.ceylon'
```

You can configure the behaviour of the plugin in the `ceylon` block.

For example:

```groovy
ceylon {
    module = "com.athaydes.maven"
}
```

The only mandatory field is `module`, which you should set to the name of the Ceylon module.

Other properties are explained in the next sections.

### Notes on default properties

The default properties of this Gradle Plugin make your Ceylon project use the default Java flat classpath
(ie. no Ceylon module isolation) because that makes it much easier to make use of several popular Java frameworks,
as well as other JVM languages, which assume free access to everything in the classpath.
  
The problem with that is that the default Ceylon module isolation is lost.

To use Ceylon defaults rather than Java's, add the following options to the `ceylon` block in your build file:
  
```groovy
ceylon {
    ...
    flatClasspath = false
    importJars = true
}
```

### Tasks

The Ceylon-Gradle Plugin adds the following tasks to your project:

* `cleanCeylon` - removes output created by the other tasks of this plugin.
* `resolveCeylonDependencies` - resolves the project's dependencies.
* `createDependenciesPoms` - creates Maven pom files for all transitive dependencies.
* `createMavenRepo` - creates a local Maven repository containing all transitive dependencies.
* `createModuleDescriptors` - creates module descriptors for all transitive dependencies.
* `generateOverridesFile` - generates the overrides.xml file.
* `importJars` - imports transitive dependencies into the `output` Ceylon repository.
* `compileCeylon` - compiles the Ceylon module.
* `compileCeylonTest` - compiles the Ceylon test module.
* `runCeylon` - runs the Ceylon module (see the `entryPoint` config property).
* `testCeylon` - runs the tests in the test module.
* `createJavaRuntime` - creates a directory with all resources required to run the Ceylon application with only the JVM.
* `fatJar` - creates a fat jar that can be used to deploy the Ceylon project more easily as a Java application.

> Lifecycle tasks such as `build` and `check` also work with the Ceylon Plugin.

Examples:

#### Running the Ceylon module with a clean environment

```
gradle cleanCeylon runCeylon
```

#### Compile the Ceylon module and show all INFO log messages

```
gradle compileCeylon --info
```

For even more output, use `--debug`.

> Another useful task that Gradle itself adds by default is `dependencies`, which prints the whole
  dependency tree of your project.

### Properties

* `get-ceylon-command`: if the project has this property, any Ceylon commands that would have been called by this plugin
  are simply printed to stdout instead of actually being called.
* `ceylon-args`: allows arguments to be provided directly to the Ceylon tool.
* `app-args`: allows arguments to be provided to the Ceylon application (only used by the `runCeylon` task).

Example usage:

```
gradle -P get-ceylon-command runCeylon
```

```
# this can be used to activate Ceylon tool options that are not supported by the Gradle Plugin 
gradle compileCeylon -Pceylon-args="--fully-export-maven-dependencies"
```

>> Notice that you can also set properties in Gradle build files, so the above could be achieved by adding
   this line to your build file: `project.ext.'ceylon-args' = '--fully-export-maven-dependencies'`

```
gradle runCeylon -Papp-args="--my-command-options other-option"
```

## Configuring the Ceylon build

Most of the configuration of this plugin is done inside the `ceylon` block.

The following properties can be set in the `ceylon` block:

* `ceylonLocation`: (optional) path to the ceylon executable. Set the `CEYLON_HOME` environment variable instead
  to make the build more portable, or do nothing if you use [SDKMAN!](http://sdkman.io/)
  as in that case, the Ceylon location will be found automatically.
* `sourceRoots`: (default: `['source']`) List of directories where the Ceylon source code is located.
* `resourceRoots`: (default: `['resource']`) List of directories where resources are located.
* `testRoots`: (default `['source']`) List of directories where Ceylon test code is located.
* `testResourceRoots`: (default `['test-resource']`) List of directories where test resources are located.
* `generateTestReport`: (default `true`) whether to ask Ceylon test to generate test reports after running tests.
* `testReportDestination`: (default `${project.buildDir}/reports`) directory to save test reports into.
* `output`: (default: `modules`) specifies the output module repository.
* `module` or `moduleName`: (**mandatory**) name of the Ceylon module.
* `testModule`: (default: same value as `module`): name of the Ceylon test module.
* `moduleExclusions`: (default: `[]`) name of the modules to remove from the compilation and runtime.
* `overrides`: (default: `'${project.buildDir}/overrides.xml'`) location to store the automatically-generated overrides.xml file.
* `mavenSettings`: (default: `'${project.buildDir}/maven-settings.xml'`) location of Maven settings file.
   If the file already exists, it is not overwritten, otherwise an appropriate file is generated (recommended).
* `flatClasspath`: (default: `true`) use a flat classpath (like in standard Java), bypassing Ceylon's default module isolation.
* `importJars`: (default: `false`) import dependencies' jar files into the Ceylon repository.
* `forceImports`: (default: `false`) use the `--force` option when importing dependencies' jar files into the 
  Ceylon repository.
* `verbose`: (default: `false`) use the `--verbose` option when invoking Ceylon commands.
* `javaRuntimeDestination`: (default: `${project.buildDir}/java-runtime`) directory to save JVM resources created 
   by the `createJavaRuntime` task.
* `fatJarDestination`: (default `${project.buildDir}`) directory to save the fat jar created by the `fatJar` task.
* `entryPoint`: (default: `${moduleName}::run`) top-level element to run when calling the `runCeylon` task 
   or the bash scripts created by the `createJavaRuntime` task.

An example configuration (using most options above) might look like this:

```groovy
ceylon {
    module = "com.acme.awesome"
    testModule = "test.com.acme.awesome"
    flatClasspath = false
    importJars = true
    sourceRoots = ['source', 'src/main/ceylon']
    resourceRoots = ['resource', 'src/main/resources']
    output = 'dist'
    verbose = true
    javaRuntimeDestination = file('target/jvm')
    entryPoint = "com.acme.awesome::start"
}
```

**See the [project samples](ceylon-gradle-plugin-tests) for working examples.**

## Integration with Eclipse

When you use both this plugin and Eclipse, you need to let Eclipse know about the resources created by this
plugin. That's very easy:

* open the `Project Properties` in Eclipse.
* go to the `Ceylon Build > Module repositories` section.
* in `Module overrides file`, enter the path to the `overrides.xml` file created by this plugin.
* if you configured this plugin to use a `flatClasspath`, check the equivalent box just below the overrides file.
* click on the `Add Maven Repository...` button.
* enter the path to the Maven `settings.xml` file created by this plugin.
* done! Just hit `OK` a few times and build the project again.

Notes:

* by default, the overrides file is saved at `build/overrides.xml`.
* by default, the Maven settings file is saved at `build/maven-settings.xml`.

## Handling transitive dependencies

All direct dependencies of your project must be declared in Ceylon `module.ceylon` file.

However, as Ceylon does not automatically resolve transitive Maven dependencies (note: this has changed in Ceylon 1.2.3 with the [`--fully-export-maven-dependencies`](https://ceylon-lang.org/documentation/current/reference/tool/ceylon/subcommands/ceylon-compile.html#option--fully-export-maven-dependencies) flag), the Ceylon-Gradle Plugin reads
the `module.ceylon` file and creates an
[overrides.xml](http://ceylon-lang.org/documentation/1.2/reference/repository/overrides/)
file that informs Ceylon what those transitive dependencies are.

The dependencies are resolved using Gradle's standard mechanism, so you can use any repository supported by Gradle.

Notice that you can exclude some particular dependency easily using Gradle. For example, to exclude dependencies with group `ch.qos.logback` (and all its transtive dependencies as well) add this to your Gradle file:

```groovy
configurations {
    all*.exclude group: "ch.qos.logback"
}
```

### Using a flat classpath VS Ceylon module system

**It is important to notice that there are 2 different ways to use the Ceylon Gradle Plugin:**

* **Flat classpath**: Java's default flat classpath, with JVM dependencies used as plain jars. This is the default.
* **Ceylon module system**: Using Ceylon's standard JBoss module system, which isolates modules' classpaths and verifies the runtime is
  consistent and not missing anything that's needed. JVM dependencies may be imported into the Ceylon repository.

Which one you should use depends on your application requirements.

* To use Java's flat classpath:

```groovy
ceylon {
    ...
    flatClasspath = true
    importJars = false
}
```

> The above is the default, so you might as well omit these parameters.

* To use the Ceylon module system:

```groovy
ceylon {
    ...
    flatClasspath = false
    importJars = true
}
```

If Ceylon complains about packages missing, this probably means the jar you are importing refers to optional Maven
dependencies' packages which Ceylon cannot guarantee will work at runtime!

To force Ceylon to accept the imports anyway, set `forceImports` to `true`:

```groovy
ceylon {
    ...
    flatClasspath = false
    importJars = true
    forceImports = true
}
```

Here's some differences between the two so you can decide which to use:

#### Flat classpath

* only one version of each module may be available to all modules at runtime.
* Java libraries may load any classpath resource and instantiate classes from other modules. Popular Java libraries such
  as Hibernate, Jersey and Guice, for example, require this behaviour.
* all packages of every module are *shared* at runtime, just like in Java. At compile-time, however, the Ceylon compiler
  can still enforce the module boundaries between different modules.
* no need to generate extra metadata to import Jars into a local Ceylon repository.

Example of importing a Maven module with coordinates `com.athaydes.groupId:module-name:1.0.0` in a Ceylon module file:

```ceylon
import "com.athaydes.groupId:module-name" "1.0.0";
```

> Notice that the module name consists of the `groupId` and the `artifactId` appended with a `:` in the middle.

#### Ceylon module system

* many versions of the same module may be used (not currently supported by this plugin's dependency resolution).
* modules boundaries are strictly enforced by Ceylon, so non-shared imports and internals of a module are completely
  invisible to other modules.
* Jars are imported into the local Ceylon repository together with a module descriptor (generated automatically by this
  plugin. Currently all dependencies are shared and non-optional).

Example of importing the same module as in the previous section, but using the Ceylon module syntax:

```ceylon
import com.athaydes.groupId.module_name "1.0.0";
```

> The module is imported into the Ceylon repository, so it must use Ceylon module name syntax: its name does not need
  to be quoted, the name is separated from the groupId with just a `.`, and the illegal character `-` is replaced
  with `_`.

### Using a Java library with many transitive dependencies

Suppose you want to use a Java library, say [SparkJava](http://sparkjava.com/),
in your Ceylon project, and you are content with using a flat classpath...
you would have a `module.ceylon` file similar to this:

```ceylon
native("jvm")
module com.athaydes.sparkweb "1.0.0" {
    import java.base "8";
    shared import "com.sparkjava:spark-core" "2.3";
}
```

Even though Spark has a lot of dependencies itself, you don't need to worry about that as Gradle will figure out
all the transitive dependencies for you.

You can see them if you want, with the `dependencies` task:

```
gradle dependencies
```

Which will print this:

```
ceylonCompile
\--- com.sparkjava:spark-core:2.3
     +--- org.slf4j:slf4j-api:1.7.12
     +--- org.slf4j:slf4j-simple:1.7.12
     |    \--- org.slf4j:slf4j-api:1.7.12
     +--- org.eclipse.jetty:jetty-server:9.3.2.v20150730
     |    +--- javax.servlet:javax.servlet-api:3.1.0
     |    +--- org.eclipse.jetty:jetty-http:9.3.2.v20150730
     |    |    \--- org.eclipse.jetty:jetty-util:9.3.2.v20150730
     |    \--- org.eclipse.jetty:jetty-io:9.3.2.v20150730
     |         \--- org.eclipse.jetty:jetty-util:9.3.2.v20150730
     +--- org.eclipse.jetty:jetty-webapp:9.3.2.v20150730
     |    +--- org.eclipse.jetty:jetty-xml:9.3.2.v20150730
     |    |    \--- org.eclipse.jetty:jetty-util:9.3.2.v20150730
     |    \--- org.eclipse.jetty:jetty-servlet:9.3.2.v20150730
     |         \--- org.eclipse.jetty:jetty-security:9.3.2.v20150730
     |              \--- org.eclipse.jetty:jetty-server:9.3.2.v20150730 (*)
     +--- org.eclipse.jetty.websocket:websocket-server:9.3.2.v20150730
     |    +--- org.eclipse.jetty.websocket:websocket-common:9.3.2.v20150730
     |    |    +--- org.eclipse.jetty.websocket:websocket-api:9.3.2.v20150730
     |    |    +--- org.eclipse.jetty:jetty-util:9.3.2.v20150730
     |    |    \--- org.eclipse.jetty:jetty-io:9.3.2.v20150730 (*)
     |    +--- org.eclipse.jetty.websocket:websocket-client:9.3.2.v20150730
     |    |    +--- org.eclipse.jetty:jetty-util:9.3.2.v20150730
     |    |    +--- org.eclipse.jetty:jetty-io:9.3.2.v20150730 (*)
     |    |    \--- org.eclipse.jetty.websocket:websocket-common:9.3.2.v2015073 (*)
     |    +--- org.eclipse.jetty.websocket:websocket-servlet:9.3.2.v20150730
     |    |    +--- org.eclipse.jetty.websocket:websocket-api:9.3.2.v20150730
     |    |    \--- javax.servlet:javax.servlet-api:3.1.0
     |    +--- org.eclipse.jetty:jetty-servlet:9.3.2.v20150730 (*)
     |    \--- org.eclipse.jetty:jetty-http:9.3.2.v20150730 (*)
     \--- org.eclipse.jetty.websocket:websocket-servlet:9.3.2.v20150730 (*)

ceylonRuntime
No dependencies
```

Luckily, you don't need to figure out all these dependencies yourself!

The auto-generated overrides.xml file for this example will be this one:

```xml
<overrides>
  <artifact groupId='com.sparkjava' artifactId='spark-core' version='2.3' shared='true'>
    <add groupId='org.slf4j' artifactId='slf4j-api' version='1.7.12' shared='true' />
    <add groupId='org.slf4j' artifactId='slf4j-simple' version='1.7.12' shared='true' />
    <add groupId='org.slf4j' artifactId='slf4j-api' version='1.7.12' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-server' version='9.3.2.v20150730' shared='true' />
    <add groupId='javax.servlet' artifactId='javax.servlet-api' version='3.1.0' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-http' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-util' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-io' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-util' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-webapp' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-xml' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-util' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-servlet' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-security' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-server' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-server' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-common' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-api' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-util' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-io' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-client' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-util' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-io' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-common' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-servlet' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-api' version='9.3.2.v20150730' shared='true' />
    <add groupId='javax.servlet' artifactId='javax.servlet-api' version='3.1.0' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-servlet' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty' artifactId='jetty-http' version='9.3.2.v20150730' shared='true' />
    <add groupId='org.eclipse.jetty.websocket' artifactId='websocket-servlet' version='9.3.2.v20150730' shared='true' />
  </artifact>
</overrides>
```

### Excluding transitive dependencies

If you don't want all of transitive dependencies to be added to your project, you can remove them by specifying them
as usual with Gradle and adding exclusions.

For example, continuing the example above, supposing that you don't want the websocket dependencies
of Jetty because you're not going to use websockets, you can declare the dependency on Spark **also** in the Gradle file,
as follows:

```groovy
dependencies {
    ceylonCompile "com.sparkjava:spark-core:2.3", {
        exclude group: 'org.eclipse.jetty.websocket'
    }
}
```

> Notice that you still must declare a dependency on `spark-core` in the `module.ceylon` file even if you add that
  dependency on the Gradle build file.
  
It's important to understand that direct dependencies should be declared in `module.ceylon`, as with any Ceylon project.
The Gradle `dependencies` should be used only to add more information to how you want transitive dependencies to be
handled, as in this example. This avoids tightly coupling your project to Gradle (once you have the overrides.xml file
generated by Gradle, your project does not need anything else from Gradle and can be run by `ceylon` as usual)
and confusion regarding where dependencies should be declared!

### Depending on other Gradle modules in the same project

You can add a dependency to another module of your Gradle project, be it a Java (or even Scala, Groovy or Kotlin)
module or another Ceylon module, using this syntax in the Gradle build file:

```groovy
dependencies {
    ceylonCompile project( ':multi-modules-project:another-module' )
}
```

Where the name of the project is `multi-modules-project`.

In the Ceylon module file, you simply refer to the name of the module (which depends on whether you're using a flat
classpath or not, as explained above).

For example, if the other module has these declarations in its Gradle build file:

```groovy
group = 'com.athaydes.gradle'
version = '4.2'
```

> notice that, with Gradle, the module name, by default, is the name of the module root directory. To change that,
  use a Gradle [`settings.gradle` file](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:settings_file).

If using a flat classpath, you would import this module in the Ceylon module file using this statement:

```ceylon
import "com.athaydes.gradle:another-module" "4.2";
```

If NOT using a flat classpath:

```ceylon
import com.athaydes.gradle.another_module "4.2";
```

## Running Ceylon code without the ceylon tool in the JVM

The `createJavaRuntime` task creates a directory containing all jars required to run your Ceylon module without using
`ceylon run`, as well as bash/bat scripts to make it trivial to start the `java` process with the Ceylon module.

It is extremely easy to do it. From the root directory of your Ceylon project, run the following commands:

#### In Linux/Mac systems

```
./gradlew clean createJavaRuntime
bash build/java-runtime/run.sh
```

#### In Windows

```
gradlew clean createJavaRuntime
build\java-runtime\run.bat
```

### Why run your Ceylon code with java instead of ceylon run?

Because you can get a 10x startup time improvement just by doing this.

Check this out (you can run the same from the [module-with-tests-sample](ceylon-gradle-plugin-tests/module-with-tests-sample)
directory):

```
  # clean, compile and create the Java runtime
$ ./gradlew clean createJavaRuntime
...

  # run the module directly with the ceylon command
  # to print the full command, run
  # ./gradlew -P get-ceylon-command runCeylon
$ time ceylon run \
    --overrides build/overrides.xml \
    --rep=aether:build/maven-settings.xml \
    --rep=modules \
    --run=com.athaydes.simple::addArgs \
    com.athaydes.simple 2 5.3 6
    
The sum of { 2.0, 5.3, 6.0 } is: 13.3

real	0m1.033s
user	0m2.459s
sys	0m0.189s

  # invoke java with the run.sh created by the Gradle Plugin
$ time bash target/jvm/run.sh 2 5.3 6

The sum of { 2.0, 5.3, 6.0 } is: 13.3

real	0m0.160s
user	0m0.173s
sys	0m0.031s
```

## Gradle project examples

For working examples of Gradle projects using the Ceylon Gradle Plugin, refer to the
[test samples directory](ceylon-gradle-plugin-tests). 


