# Services in OSGi

TODO: This chapter should be worked out in more detail, this is just the outline

## Registering a service

Using the `BundleContext`

Demo with creating an API (e.g. the `Printer`) and a service provider (e.g. `SysOutPrinter`).

Show the service repository in the webconsole

## Finding a service

Using the `ServiceTracker`

Demo with creating two service consumers (e.g. `HelloSayer` and `BonjourSayer`).

## Registration properties

Demo by adding a type property for the service provider and creating a second implementation (e.g. the `SysErrPrinter`)

## Filtering on properties

Demo by limiting 1 of the service consumers to a single type of `Printer`

Also show that there is no direct dependency from the consumer bundle to the provider bundle, only on the API bundle.
