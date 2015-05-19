# Declarative Services

TODO: This chapter should be worked out in more detail, this is just the outline

## Defining a Component

Start by changing the `SysOutPrinter` to be a component that registers itself with the bnd annotations

Add the activate and deactivate methods.

Show what DS XML is generated in the bundle

Show how the Felix SCR implementation uses this to create the component

## Referencing other services

Change the `HelloSayer` to be a component that uses a reference. First as a simple static reference.

Show that the `HelloSayer` deactivates when the `SysOutPrinter` goes away.

Now change it to be a multiple dynamic reference.

Show that it interacts with both the `HelloSayer` and the `BonjourSayer`.

## Configuration

Show that you can add configuration to the `HelloSayer` and make this component a service factory (multiple instances).

Show that you can define a filter for the reference in the configuration.

Show what metadata XML is generated in the bundle

Show what the configuration admin can do with that metadata

Show how the properties can be parsed in the activate method
