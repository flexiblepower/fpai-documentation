# Tutorial

This chapter will give a short step-by-step introduction to developing a Resource Driver, Resource Manager, an Energy App and a widget. For this example we will assume a buffer/storage appliance, in this case a battery and a dummy energy app which takes a simple decision on when to start charging the battery. We will start by getting the development environment up and running.

Code blocks are used to show some source code. All source code is also provided in the zip file. Please note that sometimes lines of code are wrapped in this document. The complete source code can be found in the EF-Pi-examples.zip file

## Workspace structure
Bndtools projects are based upon standard Eclipse Java projects.

Each bndtools workspace utilizes a general workspace-wide configuration directory called cnf. There are also setting that are shared across multiple workspaces, these can be found in the cnf.shared project. The cnf.shared project will not be explained further in this tutorial.

A file named bnd.bnd is created at the top of each Bndtools project, which controls the settings for this current project. 

The cnf directory contains files which set certain defaults:

* ext/defaults.bnd; sets General options, Java compiler options and Bnd options
* ext/repositories.bnd; contains the locations of the Bundle Repository
* demo.bndrun; is a file which defines which bundles are loaded at runtime. Note that loaded bundles are not necessarily also started. In order to start components you need to load configurations using Config Admin. We use the Felix Web Console to load configurations, which are then used to start the components.

Annotations allow to set component specific configuration, examples are shown below.

## Skeleton project
You can copy the skeleton project to have useful defaults for each new package. Note that you should <b>not</b> use File → New → Java → Java Project; but rather use this skeleton project. 
If you really want to start from scratch, use File → New → Bndtools OSGi Project. 

The implementations of the resources are based on interfaces. Some of these interfaces are already defined in the org.flexiblepower.ral.driver.* package, optionally you can also define your own interface. The implementation of your resource is placed in a specific project, based on the skeleton project e.g. `org.flexiblepower.simulation.battery`

## Example project
To illustrate the building of a resource drive, resource manager, energy app and widget, an example about a battery is used. The [Resource Driver](ResourceDriver.md) will simulate the behavior of a battery. The [Resource Manager](ResourceManager.md) will manage a battery and communicate, via the EFI, with the energy app. The [Energy App](EnergyApp.md) will show a simple energy management approach. The conculde a [Widget](Widget.md) will be constructed to show the status of the simulated battery.
  




