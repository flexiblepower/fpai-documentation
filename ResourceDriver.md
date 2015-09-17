# Resource Driver

This section explains how to write a resource driver using the battery simulation example. The resource driver forms an OSGi bundle; It is good practice to separate the interface from its implementation. This way the implementation can be changed without any impact to clients that use the interface.

## The Battery Simulation interfaces

The battery simulation implements and/or uses the following three interfaces:

* `org.flexiblepower.ral.drivers.battery.BatteryState`
* `org.flexiblepower.ral.drivers.battery.BatteryControlParameters`
* `org.flexiblepower.ral.drivers.battery.BatteryMode`

The interface definition of the BatteryState is:

```java
package org.flexiblepower.ral.drivers.battery;

import javax.measure.Measurable;
import javax.measure.quantity.Duration;
import javax.measure.quantity.Energy;
import javax.measure.quantity.Power;
import org.flexiblepower.ral.ResourceState;

/**
 * The state of the battery.
 */
public interface BatteryState extends ResourceState {
    /**
     * @return The current total capacity
     */
    Measurable<Energy> getTotalCapacity();
    /**
     * @return The power with which it charges.
     */
    Measurable<Power> getChargeSpeed();
    /**
     * @return The power with which it discharges.
     */
    Measurable<Power> getDischargeSpeed();
    /**
     * @return The amount of power that it leaks.
     */
    Measurable<Power> getSelfDischargeSpeed();
    /**
     * @return The efficiency of charging.
     */
    double getChargeEfficiency();
    /**
     * @return The efficiency of discharging.
     */
    double getDischargeEfficiency();
    /**
     * @return The minimum time during which it should be on.
     */
    Measurable<Duration> getMinimumOnTime();
    /**
     * @return The minimum time during which it should be off.
     */
    Measurable<Duration> getMinimumOffTime();
    /**
     * @return The current state of charge [0,1].
     */
    double getStateOfCharge();
    /**
     * @return The current mode.
     */
    BatteryMode getCurrentMode();
}
```

This interface is used to provide other components with state information from the battery. What is the total capacity of the battery, its charge and discharge speed, the leakage (or self discharge), the current state of charge (a number between 0 and 1 indicating the filling level) and the current BatteryMode:

```java
package org.flexiblepower.ral.drivers.battery;

/**
 * To represent the different modes a battery can be in.
 */
public enum BatteryMode {
    /**
     * When the battery is idle (no interaction with the net).
     */
    IDLE,
    /**
     * When the battery is charging (taking energy from the net).
     */
    CHARGE,
    /**
     * When the battery is discharging (providing energy to the net).
     */
    DISCHARGE;
}
```

These two interfaces are used to get information about the battery. A resource control parameter interface is used to specify the control options the resource has available. In this example the BatteryControlParameters can only return the current mode of the battery:

```java
package org.flexiblepower.ral.drivers.battery;

import org.flexiblepower.ral.ResourceControlParameters;

/**
 * The control parameters for a battery is to set its mode.
 */
public interface BatteryControlParameters extends ResourceControlParameters {
    /**
     * @return The mode
     */
    BatteryMode getMode();
}
```

## Battery Simulation bundle

Copy the skeleton project and rename it into: `flexiblepower.simulation.battery`

Within this newly created project go to the `src` folder and rename the `org.flexiblepower.example.skeleton` package into: `org.flexiblepower.simulation.battery`

This package does need to be exposed to other components (this is the implementation and not the interface). Therefore the `Export-Package` parameter in the `bnd.bnd` file must be set. As well as all flexiblepower packages must be imported.

```
-buildpath: ${default-buildpath}

Service-Component: *
Bundle-Version: 1.0.0
Import-Package: \
	org.flexiblepower*,\
	*;resolution:=optional
Export-Package:  \
	org.flexiblepower.simulation.battery
```

Create a new class (called `BatterySimulation`) that implements the `BatteryDriver` interface.

```java
/**
 * This is an example of a driver implementation for a battery simulation.
 */
@Component(designateFactory = Config.class,
           provide = Endpoint.class,
           immediate = true)
public class BatterySimulation
  extends AbstractResourceDriver<BatteryState, BatteryControlParameters>
  Implements BatteryDriver, Runnable
{
    interface Config {
        @Meta.AD(deflt = "5", description = "Interval between state updates [s]")
        long updateInterval();

        @Meta.AD(deflt = "1", description = "Total capacity [kWh]")
        double totalCapacity();

        @Meta.AD(deflt = "0.5", description = "Initial state of charge [0..1]")
        double initialStateOfCharge();

        @Meta.AD(deflt = "1500", description = "Charge power [W]")
        long chargePower();

        @Meta.AD(deflt = "1500", description = "Discharge power [W]")
        long dischargePower();

        @Meta.AD(deflt = "50", description = "Self discharge power [W]")
        long selfDischargePower();
    }
```

This code fragment of the class shows that the `BatterySimulation` not only implements the `BatteryDriver` interface, but also extends an `AbstractResourceDriver` and implements the `Runnable` interface.

The `AbstractResourceDriver` is parameterized with the types `BatteryState` and `BatteryControlParameters`, which are described in the previous paragraph.

Why Runnable needs to be implemented will be explained later on in this section.

The `BatterySimulation` class is annotated with an `@Component` annotation from bndtools. This makes this class available as a component on the EF-Pi platform. The `@Component` annotation has three attributes:
* The `designateFactory` attribute has `Config.class` as its value. This means that whenever a new `Config` is saved a new `BatterySimulation` will be started.
* The `provide` attribute indicates how this component should be registered in OSGi’s registry; in this case as a `EndPoint.class`.
* The `immediate` attribute indicates that the component should be activated immediately.

This code snippet also shows an inner `Config` interface. This allows this component to be configured from the Apache Web Console (more on this later). The `@Meta.OCD` annotation indicates that this is a OSGi Metatype interface. Within OSGi this configuration is specified in an XML file. The `@Meta` bndtools annotations make sure that is automatically created for you. OCD in `@Meta.OCD` stands for Object Class Definition which groups together a set of configurable attributes. In this case the `@Meta.OCD` annotation has a `name` attribute. This name will be shown in the Apache Web Console.

The Config interface features a number of configuration methods which influence the behavior of the simulated battery. Each method is annotated with the `@Meta.AD` (Attribute Definition) annotation. The `description` attribute within this annotation will be visible in the web console, the `deflt` attribute can be used to provide a default value for its parameter (this value can still be edited in the web console).

The next fragment shows the basic private members of the `BatterySimulation` class.

```java
	/** Reference to the 'scheduling' of this object */
	private ScheduledFuture<?> scheduledFuture;
	/** Reference to shared scheduler */
	private ScheduledExecutorService scheduler;
	/** Reference to the registration as observationProvider */
	private ServiceRegistration<Widget> widgetRegistration;
	/** Reference to the {@link TimeService} */
	private TimeService timeService;
	/** Configuration of this object */
	private Config config;
```

These variables provide references to services that are offered by the platform. Their usage will be clear from the explanation of the methods of this class.

```java
/**
 * Sets the TimeService. The TimeService is used to determine the current
 * time.
 *
 * This method is called before the Activate method
 *
 * @param timeService
 */
@Reference(optional = false)
public void setTimeService(TimeService timeService) {
	this.timeService = timeService;
}
```

This method adds a reference to the `TimeService` that is offered by the platform. Any component that needs to work with time has to have a reference to this service. Please note the `@Reference(optional = false)` annotation.

```java
/**
 * Sets a reference to a ScheduledExecutorService. This service can be used
 * to schedule tasks which implement the Runnable interface. Using a central
 * scheduler is more efficient than starting your own thread.
 *
 * This method is called before the Activate method
 *
 * @param timeService
 */
@Reference(optional = false)
public void setSchedulerService(ScheduledExecutorService schedulerService) {
	this.scheduler = schedulerService;
}
```

As is explained in the comment section of the code snippet this reference is used to schedule tasks on the EF-Pi platform.

The next fragment shows the specific private members of the `BatterySimulation` class.

```java
private static final Logger log = LoggerFactory.getLogger(BatterySimulation.class);
private Measurable<Power> dischargeSpeedInWatt;
private Measurable<Power> chargeSpeedInWatt;
private Measurable<Power> selfDischargeSpeedInWatt;
private Measurable<Energy> totalCapacityInKWh;
private Measurable<Duration> minTimeOn;
private Measurable<Duration> minTimeOff;

private BatteryMode mode;
private Date lastUpdatedTime;
private double stateOfCharge;
private BatteryWidget widget;
```

The `activate` method initializes this component and will get called after the Reference methods that were described earlier. This method is annotated with an `@Activate` notation.

```java
/**
 * This method gets called after this component gets a configuration and
 * after the methods with the Reference annotation are called
 *
 * @param context
 *            OSGi BundleContext-object
 * @param properties
 *            Map containing the configuration of this component
 */
@Activate
public void activate(BundleContext context, Map<String, Object> properties) throws Exception {
    try {
        configuration = Configurable.createConfigurable(Config.class, properties);

        totalCapacityInKWh = Measure.valueOf(configuration.totalCapacity(), KWH);
        chargeSpeedInWatt = Measure.valueOf(configuration.chargePower(), WATT);
        dischargeSpeedInWatt = Measure.valueOf(configuration.dischargePower(),WATT);
        selfDischargeSpeedInWatt = Meaure.valueOf(configuration.selfDischargePower(),WATT);
        stateOfCharge = configuration.initialStateOfCharge();
        minTimeOn = Measure.valueOf(0, SI.SECOND);
        minTimeOff = Measure.valueOf(0, SI.SECOND);
        mode = BatteryMode.IDLE;

        publishState(new State(stateOfCharge, mode));

        scheduledFuture = scheduler.scheduleAtFixedRate(this, 0, configuration.updateInterval(), TimeUnit.SECONDS);

        widget = new BatteryWidget(this);
        widgetRegistration = context.registerService(Widget.class, widget, null);
    } catch (Exception ex) {
		// When you don't catch your exception here,
		// your Runnable won't be scheduled again
        logger.error("Error during initialization of the battery simulation: " + ex.getMessage(), ex);
        deactivate();
        throw ex;
    }
}
```

The first step of this method is to acquire the configuration parameters. Next the current state (IDLE) is being published to inform the battery manager.

Often a Resource Driver will be polling an appliance for new information. When this is the case the `ScheduledExecutorService` can be used to implement the polling. The `sheduleAtFixedRate` method has the following parameters: a class that implements Runnable, an initial delay, an interval and the time unit that is used.

Here we use `BatterySimulation` class itself, this also the reason that this class implements `Runnable` as described earlier. Please note that this has been done to keep the example simple, it is better practice to schedule another class.

An resource driver can have a widget, a web interface component. The `BatterySimulation` has own to visually show the state of charge (`soc`), the total capacity of the simulated battery (`totalCapacity`) and the battery mode (`mode`) it is in. The widget must be registered as a service with OSGi. More explanation about this in the next paragraph.

To be safe, in case of an exception the battery simulation service deactivates itself.

Next to the activate, there is also an modify and a deactivate.

```java
/**
 * This method gets called after the configuration changes
 *
 * @param context
 *            OSGi BundleContext-object
 * @param properties
 *            Map containing the configuration of this component
 */
    @Modified
    public void modify(BundleContext context, Map<String, Object> properties) {
        try {
            configuration = Configurable.createConfigurable(Config.class, properties);

            totalCapacityInKWh = Measure.valueOf(configuration.totalCapacity(), KWH);
            chargeSpeedInWatt = Measure.valueOf(configuration.chargePower(), WATT);
            dischargeSpeedInWatt = Measure.valueOf(configuration.dischargePower(), WATT);
            selfDischargeSpeedInWatt = Measure.valueOf(configuration.selfDischargePower(), WATT);
            stateOfCharge = configuration.initialStateOfCharge();
            minTimeOn = Measure.valueOf(2, SI.SECOND);
            minTimeOff = Measure.valueOf(2, SI.SECOND);
            mode = BatteryMode.IDLE;

            publishState(new State(stateOfCharge, mode));
        } catch (RuntimeException ex) {
            logger.error("Error during initialization of the battery simulation: " +
     					 ex.getMessage(), ex);
            deactivate();
            throw ex;
        }
    }
```

As the activate, the modify copies the configuration to its internal variables and (re)publishes the state of the battery.

```java
/**
 * This method gets called when the service is deactivated
 */
    @Deactivate
    public void deactivate() {
        if (widgetRegistration != null) {
            widgetRegistration.unregister();
            widgetRegistration = null;
        }
        if (scheduledFuture != null) {
            scheduledFuture.cancel(false);
            scheduledFuture = null;
        }
    }
```

This method is annotated with `@Deactivate` and is called before this component gets destroyed. This allows the component to clean up before its lifecycle is ended. In this case it unregisters the widget, if available. It also cancels all actions that were planned in the future for this component.

The code snippet below shows the run() method of the `BatterySimulation` class, the place which defines the simulation behavior. It is being called at a fixed rate as has been set up in the activate method.

First the duration since the last update is being calculated. It is needed in order to determine how much energy (in watt/s) the simulation should have used to charge or discharge.
Next, depending on the running mode, the amount of charge is being based on the configuration set up. Note that discharge is defined as negative charge. The simulation takes into account the self-discharge or leakage of the battery. This is being subtracted from the amount of charge. Resulting in the charge, expressed in Watt.

By multiplying this with the duration and dividing with 1000 * 3600, the charge in KWH is being calculated. Relating this to the total capacity gives the new state of charge. An additional check is added to see if the state of charge is below zero or above one, which is not possible. In that case, the battery is empty/full and switches to idle mode. Now the new state can be published and saved as current state for the next time this method is called.

```java
    @Override
    public synchronized void run() {
        Date currentTime = timeService.getTime();
        double durationSinceLastUpdate =
			(currentTime.getTime() – lastUpdatedTime.getTime()) / 1000.0; // seconds
        lastUpdatedTime = currentTime;
        double amountOfChargeInWatt = 0;

        logger.debug("Battery simulation step. Mode={} Timestep={}s", mode, durationSinceLastUpdate);
        if (durationSinceLastUpdate > 0) {
            switch (mode) {
            case IDLE:
                amountOfChargeInWatt = 0;
                break;
            case CHARGE:
                amountOfChargeInWatt = chargeSpeedInWatt.doubleValue(WATT);
                break;
            case DISCHARGE:
                amountOfChargeInWatt = -dischargeSpeedInWatt.doubleValue(WATT);
                break;
            default:
                throw new AssertionError();
            }

            // always also self discharge
            double changeInW = amountOfChargeInWatt – selfDischargeSpeedInWatt.doubleValue(WATT);
            double changeInWS = changeInW * durationSinceLastUpdate;
            double changeinKWH = changeInWS / (1000.0 * 3600.0);

            double newStateOfCharge = stateOfCharge + (changeinKWH / totalCapacityInKWh.doubleValue(KWH));

            // check if the stateOfCharge is not outside the limits of the battery
            if (newStateOfCharge < 0.0) {
                newStateOfCharge = 0.0;
                // indicate that battery has stopped discharging
                mode = BatteryMode.IDLE;
            } else {
                if (newStateOfCharge > 1.0) {
                    newStateOfCharge = 1.0;
                    // indicate that battery has stopped charging
                    mode = BatteryMode.IDLE;
                }
            }

            State state = new State(newStateOfCharge, mode);
            logger.debug("Publishing state {}", state);
            publishState(state);

            stateOfCharge = newStateOfCharge;
        }
    }
```

A helper class is created to communicate the state of the battery, based on the state of charge and the battery mode. This helper class also has access to the configuration parameters that define the behavior of the simulated battery.

```java
    /**
     * Helper class to communicate the state of the battery
     */
    class State implements BatteryState {
        private final double stateOfCharge;
        private final BatteryMode mode;

        public State(double stateOfCharge, BatteryMode mode) {
            if (stateOfCharge < 0.0) {
                stateOfCharge = 0.0;
            }
            else if (stateOfCharge > 1.0)
            {
                stateOfCharge = 1.0;
            }

            this.stateOfCharge = stateOfCharge;
            this.mode = mode;
        }

        @Override
        public boolean isConnected() {
            return true;
        }

        @Override
        public Measurable<Energy> getTotalCapacity() {
            return totalCapacityInKWh;
        }

  --- other configuration methods are removed to safe space ---

        @Override
        public double getStateOfCharge() {
            return stateOfCharge;
        }

        @Override
        public BatteryMode getCurrentMode() {
            return mode;
        }

        @Override
        public String toString() {
            return "State [stateOfCharge=" + stateOfCharge + ", mode=" + mode + "]";
        }
    }
```

The battery manager has the possibility to control the battery simulation via the `handleControlParameters`. The only thing that can be set is a new state of the battery.

```java
    @Override
    protected void handleControlParameters(BatteryControlParameters
    controlParameters) {
        mode = controlParameters.getMode();
    }

    /**
     * Method to return the current state of the simulated battery.
     * It is being used by the widget.
     *
     * @return
     */
    protected State getCurrentState() {
        return new State(stateOfCharge, mode);
    }
```


