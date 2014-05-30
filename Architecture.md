# 1. Architecture

## Introduction

Over the years a lot of different *Demand Side Management* (DSM) approaches have been developed. Unfortunately these DMS approaches are not interoperable. A similar issue can be identified on the appliance level. Appliances provide the flexibility that is being exploited by DSM. To begin with there a lot of different appliances (washing machines, Combined Heat Power Systems, PV panels, fridges, etc.). They also use different protocols for communication (Zigbee, Z-wave, WiFi, PLC, etc.).

All this variety on both the DSM and appliance level presents *Energy Management Systems* (EMS) with a big challenge. This challenge is depicted below.

![](interaction.png)

Nowadays most EMS’es are tightly coupled to a particalar DSM approach. This results in a lock-in for consumers. A switch to another DSM approach/service almost always requires the installation of another EMS (hardware box).

The FlexiblePower Application Infrastructure (FPAI) aims to create an interoperable platform that is able to connect to a variety of appliances and support a host of DSM approaches. This way the EMS hardware does not need to be changed when a consumers switches from one service to another. At the same time the FlexiblePower Application Infrastructure makes it easier for service providers to introduce new services, since they do not have to provide the EMS hardware to their consumers to go with it.

## High-level design
![](hourglass.png)

The FlexiblePower Application Infrastructure is the connecting part between appliances at the home of the consumers at one side, and the (smart) energy grid on the other side. Appliances of different vendors can implement functionality which can be used to choose when and maybe how to start and use certain energy consuming or energy providing appliances. Developers of FPAI are responsible for providing functionality which connects these parts. Protocols are used to communicate with appliances within appliance drivers. 

The App Store and Remote Management Interface are still under development and thus not discussed in this tutorial.

For you as developer it is important to know how the infrastructure works:

* A Resource Manager receives a State from a Resource Driver. 
* This State is obtained by the Resource Driver using information sent by or polled from the appliance(s). 
* The State is converted by the Resource Manager into a Control Space. 
* The Resource Manager sends this Control Space to the Energy App. A Control Space defines the freedom in which the appliance can be started, and how much energy is consumed or produced when started. 
* The Energy App then provides an Allocation, which states when each appliance should be started, which is a time within the Control Space. 
* A Resource Manager creates Control Parameters based on this, which define when the appliance should start. The Control Parameters are sent to the Resource Driver.
* The Resource Driver actually starts the appliance.

A dashboard can show controls and information about the current state of your appliances in the form of Widgets. Each Widget shows information for an appliance. It is possible to have multiple Widgets per appliance. When run the Dashboard is currently shown at http://localhost:8080/


## Components and widgets

![](component_overview.png)