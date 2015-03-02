# Setting up your IDE

## Introduction
In this tutorial we will explain how you to set up your development environment.

## Installing the Java Development Kit
First of all, the Java Development Kit (a.k.a. the JDK) needs to be installed. The FPAI is developed in Java  version 1.6 (also refered to as version 6), so at least version 1.6 of the JDK needs to be present. Later versions also work fine, so you can just install the latest version for your operating system.

To check if you have installed the JDK you can execute the command `javac -version` on the command line. Windows users can access the command line through Start menu, press Run, type `cmd` and click the OK button. If the JDK is installed you should see the version number. If the JDK is not installed you should get a warning indicating that the command could not be found.

Windows and Mac users can download the JDK from [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html "Oracle"). Linux users can install Java through their package manager. OpenJDK also works fine. For example, Ubuntu users can install the JDK with the following commands:

```
sudo apt-get update
sudo apt-get install openjdk-7-jdk
```

## Installing Eclipse
The preferred IDE for developing apps for FPAI is Eclipse, currently at version Luna. The FPAI heavily uses bnd for developing OSGi-bundles, and Eclipse has an excellent plug-in for bnd.

The latest version of Eclipse can be downloaded from [http://www.eclipse.org/downloads/](http://www.eclipse.org/downloads/). The Standard Eclipse distribution or the Eclipse IDE for Java Developers version is preferred. Be careful to select the right version for your platform. Eclipse is provided as a zip archive. It doesn't have an installer. Just extract Eclipse to a location which is convenient for you.

For more information see [http://www.eclipse.org](http://www.eclipse.org "Eclipse website").

## Installing Git
There are several ways can use Git. You could use the GitHub Desktop Client ([Windows](https://windows.github.com) or [Mac OS X](https://mac.github.com)), Git from command line (Git Bash) or the [Eclipse EGit plug-in](http://www.eclipse.org/egit/). In this tutorial we will assume you use Git Bash. You can download it for your platform on [http://git-scm.com/downloads](http://git-scm.com/downloads) (or use the package manager for your Operating System).

For more details on using Git we recommend reading the [Git Pro book](http://git-scm.com/book). You can read it online for free.

## Cloning the example repository
In this guide we will use the `fpai-examples` repository. This repository contains simple examples of Apps for the FPAI. You can find the repository [on GitHub](https://github.com/flexiblepower/fpai-examples).

First of all, there are two ways of using the Git repository. If you only want to fiddle around with the examples, you can just check out the repository with HTTPS. This way you don't have to create a Github account. Changes you make to the code will only stay on your machine.

You can also decide to fork the repository. This way you make a private copy of the repository on Github. You can push your changes to Github and share them with other users. This requires you to create a Github account, [upload your SSH keys](https://help.github.com/articles/generating-ssh-keys/), [fork the repository](https://help.github.com/articles/fork-a-repo/) and checking out the repository over SSH. For this tutorial it doesn't really matter which method you choose.

On the right side of the [repository page](https://github.com/flexiblepower/fpai-examples) you can find the clone URL. When you don't have a Github account this should be `https://github.com/flexiblepower/fpai-examples.git`. Now we have to open up Git Bash and go the directory where you want your source code. Let's assume you have created this directory at `C:\Code\FPAI`. Since Bash is designed for Linux, you can use forward shlashes instead of back slashes and use `/c/` instead of `C:\`. You can change the directory with the `cd` (change directory) command. In order to go to the right directory you have to type in the following command and press enter:

```
cd /c/Code/FPAI
```

Now we have to clone the Git repository into this directory. You can use the `git clone` command with the clone URL:

```
git clone https://github.com/flexiblepower/fpai-examples.git
```

Git will create a directory named `git-examples` inside the `C:\Code\FPAI` directory with the source code.

In the FPAI projects we use [git submodules](http://git-scm.com/docs/git-submodule) to share some files with several repositories. In order to be able to use these files you have first have to go inside your repository directory, and then execute two commands for initializing the submodule:

```
cd fpai-examples
git submodule init
git submodule update
```

It might take some time to download these additional files. When these task are done you should also have a `cnf.shared` directory in your repository.

## Starting Eclipse
Now it is time to start Eclipes. The first thing Eclipse asks is the location of your workspace. For the FPAI, each repository is also a workspace. So select the directory you just checked out (`C:\Code\FPAI\fpai-examples`).

When you start Eclipse for the first time it will show you the Welcome page. You can close it.

**Tip:** Are you using a lot of workspaces? When Eclipse starts, it asks you which workspace you want to use. Eclipse only remembers the last five workspaces you have used. Once you have opened up a workspace, you can increase this number by going to 'Window', 'Preferences' and than 'General', 'Startup and Shutdown'. Here you can manage the known workspaces and change the number of recent workspaces to remember.

## Installing Bndtools
The Eclipse plugin for bnd, called Bndtools, can be obtained from the Eclipse Marketplace. In Eclipes, go to `Help` and then to `Eclipse Marketplace...`. Search for `Bndtools` and click the `Install` button. During installation you will get a warning complaining about unsigned content. You may ignore that warning. After installation, restart Eclipse.

For more information see [http://bndtools.org](http://bndtools.org). There is also a general tutorial available at [http://bndtools.org/tutorial.html](http://bndtools.org/tutorial.html).


## Importing the projects
The repository you checked out already contains some projects. We have to tell Eclipse to search for those repositories first. You can do this by going to `File` and than `Import`. Select `Existing Projects into Workspace` from the `General` directory and click the `Next` button.

Click on the `Browse` button on top of the screen. It will automatically go to the your workspace directory. Click `OK` to select your workspace directory. Select all the projects and click `Finish`. Eclipse will add the projects to the Package explorer on the left of the screen.

Finally, we have to select the Bndtools perspective. A perspective is a configuration of the Eclipse user interface which is made for a specific tasks. You can select a perspective by clicking `Window`, `Open perspective` and than `Other...`. Double click on `Bndtools` to open the Bndtools perspective. You can close the welcome window of eclipse, and tick the box not to show again.