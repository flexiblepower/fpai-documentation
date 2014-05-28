# Setting up your IDE

## Introduction
In this tutorial we will explain how you to set up your development environment.

## Installing the Java Development Kit
First of all, the Java Development Kit (a.k.a. the JDK) needs to be installed. The FPAI is developed in Java  version 1.6 (also refered to as version 6), so at least version 1.6 of the JDK needs te present. Version 1.7 (or version 7) works also fine. At the moment we would not recommend using Java 8.

To check if you have installed the JDK you can execute the command `javac -version` on the command line. Windows users can access the command line through Start menu, press Run, type `cmd` and click the OK button. If the JDK is installed you should see the version number. If the JDK is not installed you should get a warning indicating that the command could not be found.

Windows and Mac users can download the JDK from [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html "Oracle"). Linux users can install Java through their package manager. OpenJDK also works fine. For example, Ubuntu users can install the JDK with the following commands:

```
sudo apt-get update
sudo apt-get install openjdk-7-jdk
```

## Installing Eclipse
The preferred IDE for developing apps for FPAI is Eclipse, currently at version Kepler. The FPAI heavily uses bnd for developing OSGi-bundles, and Eclipse has an excellent plug-in for bnd.

The latest version of Eclipse can be downloaded from [http://www.eclipse.org/downloads/](http://www.eclipse.org/downloads/). The Standard Eclipse distribution or the Eclipse IDE for Java Developers version is preferred. Be careful to select the right version for your platform. Eclipse is provided as a zip archive. It doesn't have an installer. Just extract Eclipse to a location which is convenient for you.

For more information see [http://www.eclipse.org](http://www.eclipse.org "Eclipse website").

## Installing Git
There are several ways can use Git. You could use the GitHub Desktop Client ([Windows](https://windows.github.com) or [Mac OS X](https://mac.github.com)), Git from command line (Git Bash) or the [Eclipse EGit plug-in](http://www.eclipse.org/egit/). In this tutorial we will assume you use Git Bash. GitHub provides an [excellent guide](https://help.github.com/articles/set-up-git) on installing and using it.

For more details on using Git we recommend reading the [Git Pro book](http://git-scm.com/book). You can read it online for free.

## Checking out the example repository


## Installing Bndtools
The Eclipse plugin for bnd, called bndtools, can be obtained from the Eclipse Marketplace (Help →
Eclipse Marketplace…), see Figure 2. During installation you will get a warning complaining about
unsigned content. You may ignore that warning. After installation, restart Eclipse.
This should provide all prerequisites for compiling and running the FPAI-framework which we will
introduce next.

For more information see [http://bndtools.org](http://bndtools.org). There is also a general tutorial available at [http://bndtools.org/tutorial.html](http://bndtools.org/tutorial.html).
