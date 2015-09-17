# Design Philosophy

To design a distributed runtime, where on each node you can dynamically install new components, it is very important to have a clear design strategy. Different component should not interfere with each other and the whole runtime should be as robust as possible. To achieve this, a couple of strategies have been employed.

## Component separation with clear API

Each component should be able to work with as few dependencies as possible, which means that the components are 'loosely coupled'. Each interaction between components should be achieved through splitting it up into 3 parts:

* The API, which only contains the interface(s) and relevant (immutable) data objects.
* The provider that implements the interface.
* The consumer that uses the interface.

This way the consumer should never see the real implementation of the component, meaning that it can be exchanged for other implementations. Also it also makes it virtually impossible to create 'spaghetti code', where all the components are linked to all the other components. It also forces the developer to think clearly about the functionality that he is creating and if it needs to be available to other components in the system.

### An example

I would like to develop a forecasting component for PV panels and I need the weather forecasts. Of course I could simply implement it in my own component, but the weather forecasts could be useful for other components and it provides a separate functionality. This is a good reason to put it in its own component with a clear API.

First I define the API by defining a `WeatherForecastService`, which probably looks something like this:

```
public interface WeatherForecastService {
    /**
     * Gets the {@link WeatherForecast} for the given period. Notice that the implementation may
     * round of the period to the closest timeframe it supports. Also some averaging may take
     * place to match the timeframe.
     *
     * @param from The starting date of the period
     * @param timeframe The duration of the period.
     * @return The forecast for the given period.
     */
    WeatherForecast getForecast(Date from, Measurable<Duration> timeframe);
}
```

Now when I create an implementation of this service, the implementation can be made private for that component, but it can be registered in the service repository as a `WeatherForecastService`. This means that the driver for the PV Panels can look for service that provides the weather forecast. This has several advantages, despite it being a little bit more work:

- *Seperation of concerns*: When developing each component, you can really focus on a small piece of functionality without having to worry about the impact on other components. The only thing you have to keep in your mind is the contract you have specified in the API.
- *Upgradability*: Because the components are clearly separated, it is much easier to update one of the components without having to rebuild and redistribute the whole application.
- *Reuse*: Reusing a separate bundle is much easier than having to copy code.
- *Manageability*: With a framework that can monitor and control the lifecycle of services and components, it is much easier te diagnose problems in a large complex system. When everything is interwoven with each other, debugging becomes very hard to do.  

## Thread separation

When calling functions directly of components from other bundles, you give control of your thread to that other bundle. In many cases this is not a big issue, but in a more complex system such as the EF-Pi, we have run into trouble when the smart energy application does a lot of calculations based on an update from the driver. Then you want to make sure that the driver can still function and doesn't have to wait for the messages to be handled.

In the EF-Pi, the messaging framework and the use of scheduling in the `FlexiblePowerContext` makes sure that each bundle has its own thread. When a bundle then makes the mistake of creating a long-lived task that doesn't return, the scheduling for all the other bundles is not affected and in the management view you can see which thread has stalled. 

## Immutable objects

When doing messaging between component on possible different threads, you are handing over object in memory. These object should always be immutable. This makes it much easier to reason over these objects (they can not change) and there is no locking needed when you want to make a change, because changing the object means making a new (immutable) copy.

