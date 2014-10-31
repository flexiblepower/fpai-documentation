# Observation Framework

To be able to monitor systems connected to the FPAI, an observation framework has been introduced. This framework makes it possible to add software components that can publish or subscribe to observations of a certain type within a single FPAI node.

There are two roles defined in this framework; the `ObservationProvider` and the `ObservationConsumer`. These roles can be filled by any component that is an application on top of FPAI. See this figure for an overview.

![Overview of the Observation Framework](FPAI/Observation-Scope.png)
*Overview of the Observation Framework*

The concept is similar to a [publish/subscribe patter](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) where the messages are observations of a (possible real-world) object. The `ObservationProvider` is a publisher to which `ObservationConsumers` can register itself. After being registered the `ObservationConsumer` received `Observation` objects through its `consume` method. This relationship is many-to-many, so several consumers can subscribe to the same provider and one consumer can subscribe to more providers than one. 

## Observer vs. Observable

There is an important distinction between the observer and the observable. The monitoring framework in FPAI will try to facilitate the two concepts. Observations are made by observers (sensors). Observations may be provided with an identification of the observer (sensor), e.g. in a straightforward triple of: timestamp, sensor-id, value. Attribution of a certain observation to an attribute of an observable object may be done in ‘central’ processing.

## Observation object

To send the tuple of the timestamp and the observation data, the `Observation` object must be used. The data itself must be a immutable Java data object with getter for each of the attributes. This information is used both for meta-typing (see next section) and for parsing the data when the typing information is lost.

The `Observation` object also provides a method to read the observation data as a `Map<String, Object>` using the `getValueMap()` method.

## ObservationProvider registration

For the observation framework a [whiteboard pattern](www.osgi.org/wiki/uploads/Links/whiteboard.pdf) is used. In this case the `ObservationProvide` will register itself in the service repository, such that it can be found by any `ObservationConsumer`. During the registration a few properties should be added:

 - `org.flexiblepower.monitoring.observationOf` Describes the observable object that is being observed.
 - `org.flexiblepower.monitoring.observedBy` Describes the sensor or software component that is doing the observing.
 - `org.flexiblepower.monitoring.type` The classname of the data object that contains the observation.
 - `org.flexiblepower.monitoring.type.<name>` For each attribute of the data object, the `<name>` is replaced by the name of the attribute (as will be returned by the `Observation.getValueMap()` method) and the value is the type description, usually the classname.
 - `org.flexiblepower.monitoring.type.<name>.unit` (optional) Describes the unit in which the attribute should be interpreted. This is only relevant for raw values, like doubles or integers.
 - `org.flexiblepower.monitoring.type.<name>.optional` (optional) Describes if the attribute is optional. By default the consumer may assume that all attributes are available each time, but by setting this to true it can indicate otherwise.

To make it easy to generate these properties, the `ObservationProviderRegistrationHelper` can be used. For example:

```java
@Activate
public void activate() {
  // Other activation stuff...
  registration = new ObservationProviderRegistrationHelper(this).observationOf("some sensor")
                                                                .observationBy(getClass().getName())
                                                                .observationType(SensorState.class)
                                                                .register();
}

@Deactivate
public void deactivate() {
  // Other deactivate stuff...
  registration.unregister();
} 
```

## ObservationConsumer usage

Any object that implements the `ObservationConsumer` can easily find any `ObservationProvider` in the service registry. This can be done using the [BundleContext.getServiceReferences()`](http://www.osgi.org/javadoc/r4v43/core/org/osgi/framework/BundleContext.html#getServiceReferences(java.lang.Class, java.lang.String)) or using declarative services. For example, how to get all providers or a specific observation type.

```java
@Reference(dynamic=true, multiple=true, optional=true, target="(org.flexiblepower.monitoring.type=org.flexiblepower.example.SensorState)")
public void addObservationProvider(ObservationProvider<SensorState> provider, Map<String, Object> properties) {
    // Do something with it!
}
```