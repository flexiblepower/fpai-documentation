# FlexiblePower Application Infrastructure

This section describes the different parts that make up the FlexiblePower Application Infrastructure (FPAI). This includes:

 - [Resource Abstraction Interface](RAI): The interfaces that define what a `ResourceDriver`, `ResourceManager` and `ControllerManager` are and how they should be implemented by appliance drivers or energy applications.
 - [Measurable Objects](Measure): Describes the framework to describe measurable values with quantities.
 - [Runtime Services](Runtime): The services that are provided by default through the runtime project.
 - [Messaging Framework](MessagingFramework): The framework that makes it possible to communicate asynchronously between components.
 - [Observation Framework](ObservationFramework): The framework that makes it easy to publish observations (such as from sensors), including metadata that describe the type of observations that are possible.
 - [Widgets](Widgets): The widgets make it possible to easily provide a user interface for any component on the dashboard.

## Core Bundles

In the [fpai-core repository](https://github.com/flexiblepower/fpai-core) you will find all the bundles for the FPAI platform.
This repository is really the core of the FPAI; it contains no Apps, such as device drivers, protocol drivers or smart grid 
applications.

###flexiblepower.api
This bundle contains all interfaces and data objects that make up the FPAI specification. This includes the interfaces for 
runtime services (provided by flexiblepower.runtime), the RAI, the messaging framework, the observation framework and 
common data objects that are used throughout the FPAI.

###flexiblepower.api.ext
This bundle contains abstract implementations for the RAI, such as the `AbstractResourceManager` and `AbstractResourceDriver` 
that make it easier to implement drivers. It also includes definitions for specific drivers (e.g. for a refrigerator or
dishwasher). More information on using these is shown in the [tutorial](../ResourceDriver#battery-simulation-bundle).

###flexiblepower.api.efi
This bundle contains interfaces and classes for data objects for the Energy Flexibility Interface (EFI). The EFI is the
standard RAI that is used in the FPAI. More information on the EFI can be found [here](../EnergyFlexibilityInterface).

###flexiblepower.api.efi.utils
This bundle contains utility classes which help Energy Apps interpret messages from the Energy Flexibility Interface (EFI).

###flexiblepower.runtime
This project generates 2 bundles that can both be used as a runtime environment of the FPAI. It provides implementations of
the (runtime services)[Runtime] as defined in the API.

The first bundle is the normal runtime (called `flexiblepower.runtime`). This implements the `TimeService` and 
`ScheduledExecutorService` in the normal way, using the system clock. The second bundle is the simulation runtime (called 
`flexiblepower.runtime.simulation`). This implements these services using a simulated clock, which can be controlled using a
widget.

###flexiblepower.runtime.test
This bundle contains a couple of integration tests and integration tests for the FPAI runtime. This bundle should never be used
in a deployment setting.

###flexiblepower.ui
This bundle contains the web user interface, including a dashboard with widgets.

###flexiblepower.ui.resourceinfo
This bundle adds a page to the UI that displays information on appliances connected to the FPAI.

###flexiblepower.felix.webconsole.plugins
This bundle contains plugins for the Felix Web console. They provide useful tools for FPAI developers or administrators.

At this moment it only includes a `ConnectionManagerPlugin` that provides a graphical representation of the 
[[endpoints|MessagingFramework#endpoint]] and the connections between them.
