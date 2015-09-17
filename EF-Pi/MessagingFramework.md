# Messaging Framework

In order to be able to communicate between objects, there is a Messaging Framework implemented in the EF-Pi-core. This provides a way to send any object from bundles to other bundles asynchronously, with asynchronous handling as well.

The implementation is found at `org.flexiblepower.messaging` in `EF-Pi-core`.

A component, like ResourceDrivers, ResourceManagers, EnergyApps etc., can use this framework by providing an `Endpoint` using the

Communication wiring is done using a messaging abstraction, implemented in the `org.flexiblepower.messaging` package. This is implemented using annotations:
Any component, like an implementation of a ResourceDriver or ResourceManager, can have an annotation `@Ports` which can contain multiple `@Port` definitions, which have a `name`, `cardinality`, `sends`, and `accepts` parameter; the latter specify which objects are accepted by this object.
They then must implement a MessageHandler that accepts the object(s). The enum Cardinality (SINGLE, MULTIPLE) can be given to specify whether it support a single or multiple connections. Exact details are specified in the tutorial that follows later. Note that the usage of the `ResourceId` property is depricated since the 14.10 release.

Under water it is implemented as follows: the `ConnectionManager` object is responsible for keeping the administration. It holds the interfaces for retrieving all `ManagedEndpoint`s. A `ManagedEndpoint` has a port, in the form of the `EndpointPort` class. This class is instantiated by the service using the annotations as explained above. It holds the `PotentialConnection`s, which possibly connects two `EndpointPort`s.

Within EF-Pi the `PotentialConnection`s are shown in the Felix Webconsole, as explained in [Working with OSGi](../OSGi.md) chapter. It has an EF-Pi plugin called `ConnectionManager` under “Main” → “EF-Pi: ConnectionManager”. The ConnectionManager graphically displays the different components (that were started previously) as rectangles. This shows all `EndPoint`s, with their `EnpointPort`s. When an `EndPoint` has a Port that sends and receives the objects another `EndPoint` knows how to receive and send, they have a possible connection and are shown as yellow in the UI:

![Possible connections](ConnectionManager-1.png)

Connected connections are shown as green, whilst possible connections that are impossible to make, due to cardinality constraints, are shown in red.

![Connected connections](ConnectionManager-2.png)

You can press the Autoconnect button to connect all connections that are unambiguous to make (i.e. not depending on cardinality, and not able to connect to multiple ports).

The connections will now show up green, which means that the connection has been realized and messages can be exchanged between the components. These messages are also logged in the eclipse console.