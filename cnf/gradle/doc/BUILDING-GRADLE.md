# Introduction<a name="Introduction"></a>

This bnd/bndtools workspace can be built using Gradle.

The Gradle build will include all projects in the build that are:
* bnd projects   : A bnd project is a directory directly below the root
                   project with a "bnd.bnd" file.
* gradle projects: A gradle project is a directory directly below the root
                   project with a "build.gradle" file.


# Installing Gradle

## On The System

Obviously, Gradle must be installed on the system before the workspace can be
used with Gradle.

This description assumes a Linux machine. Details may vary on other OSes.

* Download Gradle from http://gradle.org.
* Unpack it, and put it in ```/usr/local/lib```.
* Assuming Gradle 1.10 was downloaded, there will be a
  directory ```/usr/local/lib/gradle-1.10```.
* Put the Gradle executable ```/usr/local/lib/gradle-1.10/bin/gradle``` on
  the search path by linking to it from ```/usr/local/bin```:

```
ln -s /usr/local/lib/gradle-1.10/bin/gradle /usr/local/bin/
```


## In The Workspace

Gradle can be installed in the workspace so that the workspace can be built on
systems that do not have Gradle installed (like build servers).

The procedure is:
* Open a shell.
* Go to the workspace root directory.
* Assuming Gradle is properly installed on the system, run:

```
gradle wrapper
```


# Configuring The Gradle Daemon

Startup times of a Gradle build can be much improved by using the Gradle daemon.

The Gradle daemon works well when the Gradle build scripts are not changed,
which makes it well suited to regular development but not for build servers.

The daemon can be easily setup by adding the following line
to ```~/.gradle/gradle.properties```:

```
org.gradle.daemon=true
```


# Understanding The Build

## Gradle Build Setup Diagram

[This diagram](cnf/gradle/template/gradle.svg) shows the build setup,
like shown for the Java Plugin in the Gradle User Guide.

The arrows show execution flow, so the dependencies are the other way around.

The dotted blue arrow shows a convenience flow/dependency; running
```gradle jar``` will build all bundles and jars in all included
projects - bnd projects as well as java projects.

The red arrows show flows/dependencies on dependent projects. For example:
the ```compileJava``` phase of a project is dependent on the ```bundle```
phase of another project if the latter project is on the build path of the
former project.

The dotted green arrows depict tasks that are disabled by default.


## Build Flow<a name="BuildFlow"></a>

The build has the following flow:

1. Gradle locates the root project by looking for a directory with
   a ```settings.gradle``` file. It does so by first looking in the current
   directory and then searching upwards in the directory tree.

2. From the root project the ```settings.gradle``` file is loaded,
   which will instruct Gradle to include all bnd projects and Gradle projects
   (see [Introduction](#Introduction) for an explanation).

3. From the root project Gradle loads the ```build.gradle``` file, which will:

   1. Load the default build settings.

      The default build settings are used to:
      1. Configure the build itself.
      2. Configure tasks of the root project.
      3. Configure tasks of the included projects.

   2. Load the overrides of the build settings (from
      the ```cnf/gradle/custom/build-settings-overrides.gradle``` file).

   3. Setup the build dependencies.

      All jars and bundles that are located in ```cnf/plugins``` are considered
      to be build dependencies. Additional build dependencies are specified in
      the ```cnf/build.properties``` file:
      1. All ```*.url``` and ```*.location``` settings are build dependencies.
      2. A ```*.url``` setting has priority over the
         corresponding ```*.location``` setting, which means that
         the ```example.location``` setting will be ignored if ```example.url```
         is also specified.
      3. An ```example.url``` setting will make the build script download
         the ```example``` build dependency from the specified URL into
         the ```cnf/cache/gradle``` directory and add it to the build
         dependencies.
      4. An ```example.location``` setting will make the build script add the
         specified location (path) to the build dependencies.

   4. Initialise the bnd workspace and load the workspace defaults.

   5. Apply the build template.

      1. Setup defaults for all projects.
      2. Apply build settings overrides for all projects that have
         a ```build-settings-overrides.gradle``` file.
      3. Apply build customisations to all projects (from
         the ```cnf/gradle/custom/all-projects.gradle``` file).
      4. Apply build customisations to all non-bnd projects (from
         the ```cnf/gradle/custom/all-nonbndprojects.gradle``` file).
      5. Apply the bnd build setup to all bnd projects.
      6. Apply build customisations to all bnd projects (from
         the ```cnf/gradle/custom/all-bndprojects.gradle``` file).
      7. Apply build customisations to all included projects (from
         the ```cnf/gradle/custom/all-subprojects.gradle``` file).
      8. Setup the root project.
      9. Apply build customisations to the root project (from
         the ```cnf/gradle/custom/rootproject.gradle``` file).

4. Gradle now resolves the build setup.

5. Gradle will build the project.


# Build Options

## Findbugs

* ```findbugs``` must be explicitly specified as a task to run (on the
  command line) for the ```findbugsMain``` task to be enabled.
* ```findbugstest``` must be explicitly specified as a task to run (on the
  command line) for the ```findbugsTest``` task to be enabled.
* Set the ```CI``` property (```-PCI``` on the command line) to generate XML
  reports instead of html reports.


# Customising The Build

The build be can easily customised by putting overrides in any of
the ```cnf/gradle/custom/*.gradle``` build script files.

Also, any project can - on an individual basis - override build settings by
placing a ```build-settings-overrides.gradle``` in its root directory.

Make sure to read the [Build Flow](#BuildFlow) section to pick the correct
build script file.
