# Building

The most convenient way to build EF-Pi during development is to let Eclipse do the work. However, it is also possible to build EF-Pi from the command line using Gradle. Gradle is an advanced general puprose build tool. For more information see [https://gradle.org/](https://gradle.org/ "https://gradle.org").

## Gradle Wrapper

EF-Pi uses **Gradle version 2.5**. Since there are many different versions of Gradle, most versions make use of Gradle Wrapper. This is a small program that will install the right version of Gradle, if this is not done yet.

On Windows, you can start Gradle by executing the `gradlew.bat` file:

```
> gradlew.bat
```

On Linux, you can make use of the `gradlew` file:

```
./gladlew
```

The first time you use Gradle Wrapper it will download and install the right Gradle version locally.

## Tasks

The Gradle Wrapper needs to be followed by the name of a task. In order to get an overview of all available tasks, you can use the task `tasks`. For example (on Linux):

```
./gradlew tasks
```

This will show you all available tasks. For example, to generate the jar files using Gradle, you need to use the task `jar`. For example (on Linux):

```
./gradlew jar
```

For more information see the [https://gradle.org/guides/#getting-started](https://gradle.org/guides/#getting-started "Gradle tutorials").

## Building the runtime environment

When EF-Pi needs to run in a stand-alone fashion, you need to build the runtime environment. This can be done with the task `generateRuntime`. This task will generate a zip archive with all the resources to run EF-Pi without Eclipse. This way, EF-Pi can for example be deployed on a Linux server or a Raspberry Pi. The archive contains a `run.sh` file in order to run it an Linux machine, and a `run.bat` file to run it on a Windows machine.

On Linux machines the zip will be default be saved in the directory `/mnt/fan-ci/builds`. On Windows the default directory is `C:\temp\builds`. It will be saved in a directory structure. These locations can be changed in the `cnf/gradle/init.gradle` file inside the workspace.

## Building the complete distribution
With the task `distribute` a complete distributed can be generated, including source files, javadoc, the runtime environment and some convenient zip archives. This can be useful for continuous integration purposes.