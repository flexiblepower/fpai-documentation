# Runtime Services

The `flexiblepower.runtime` bundle provides a couple of default services. The `TimeService` and `ScheduledExecutionService` should be used by any application that wants to do some work at regular intervals, such as `ResourceDriver`s. Using these 2 services makes it possible to replace the runtime bundle with the `flexiblepower.runtime.simulation` bundle, which provides a simulated clock that can run tasks at an increased rate.

## ConnectionManager

The runtime also provides an implementation of the `ConnectionManager`. This service is responsible for detecting endpoints for the messaging framework and provides an API to determine which connections should be activated and which should not be activated. For more information on this implementation, see [this section](MessagingFramework.md).

## TimeService and ScheduledExecutionService 

The `TimeService` interface should always be used to get the current time. This makes it possible to use the component in simulations, where the system time should not be used, but the simulated clock.

The interface provides two methods: the `getTime()` method returns a new `Date` object that contains the current time (as a replacement for `new Date()`) and the `getCurrentTimeMillis()` method returns the number of milliseconds since the unix epoch (as a replacement for `System.currentTimeMillis()`).

The interface of the `ScheduledExecutionService` is defined in the [Java API](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html) and should be used if executing tasks in the future (or at a regular interval) is needed. 

For example, to schedule that the run method should be called every 5 seconds and then to do someting that requires the current time:

```java
private ScheduledExecutorService scheduler;

@Reference
public void setScheduledExecutorService(ScheduledExecutorService scheduler) {
    this.scheduler = scheduler;
}

private TimeService timeService;

@Reference
public void setTimeService(TimeService timeService) {
    this.timeService = timeService;
}

private ScheduledFuture<?> scheduledFuture;

@Activate
public void activate() {
	// Other activation stuff
	scheduledFuture = scheduler.scheduleAtFixedRate(this, 0, 5, TimeUnit.SECONDS);
}

@Deactivate
public void deactivate() {
	scheduledFuture.cancel(false);
}

@Override
public void run() {
    Date now = timeService.getTime();
    log.debug("It is now {}", now);
    // Do something useful with the time
} 
```

Please note that the `shutdown`, `shutdownNow` and `awaitTermination` methods will throw a `SecurityException`, because the lifecycle of this service is managed by OSGi.

### TimeUtil

Since timestamps (using `Date` objects) and duration (using `Measurable` objects) are a central part of the FPAI framework, the API also provides a `TimeUtil` class to combine these two.

```java
Date now = timeService.getTime()

Date oneHourInTheFuture = TimeUtil.add(now, Measure.valueOf(1, HOUR));
Date oneHourInThePast = TimeUtil.subtract(now, Measure.valueOf(1, HOUR));
Measurable<Duration> twoHours = TimeUtil.difference(oneHourInThePast, oneHourInTheFuture);
```
