# Resource Manager

The resource manager does not offer services to other components. That means that it is implemented as one OSGi service.

## Resource Manager implementation bundle

Copy the skeleton project and rename it into: `flexiblepower.battery.manager`.
Within this newly created project go to the src folder and rename the `org.flexiblepower.example.skeleton` package into: `org.flexiblepower.battery.manager`.
Make sure this new package is referred to by the `Export-Package` property in the bnd.bnd file. This package needs to be exported by this bundle, since it offers a resource manager services, more specific the battery manager service, that can be used by other components. This project also makes use of the `flexiblepower.api.efi` bundle, so it needs to be included on the `-buildpath` property in the `bnd.bnd` file. The file should look as follows.

```
-buildpath:  \
	${default-buildpath},\
	flexiblepower.api.efi

Bundle-Version: 1.0.0.${qualifier}

Service-Component: org.flexiblepower.battery.manager.*
Export-Package:  \
	org.flexiblepower.battery.manager
```

Create a new class `BatteryManager` that extends `AbstractResourceManager` and implements the `BufferResourceManager` interface.

```java
@Component(designateFactory = Config.class, provide = Endpoint.class, immediate = true)
public class BatteryManager
	   extends AbstractResourceManager<BatteryState, BatteryControlParameters>
       implements BufferResourceManager
{
    private static final Logger log = LoggerFactory.getLogger(BatteryManager.class);

    @Meta.OCD
    interface Config {
        @Meta.AD(deflt = "BatteryManager", description = "Unique resourceID")
        String resourceId();
    }
```

The `AbstractResourceManager` is parameterized with the `BatteryState` and the `BatteryControlparameters`, which are introduced in paragraph 3.5 about the Resource driver.

The `BatteryManager` has an `@Component` annotation. In this case this component will be registered in the OSGI registry as a `Endpoint.class`.

The code snippet also show the inner Config interface. The link to the configuration parameters. The `BatteryManager` class only has a `resourceId` as a configurable attribute.

```java
@Port(name = "controller",
      accepts = { BufferAllocation.class, AllocationRevoke.class },
      sends = { BufferRegistration.class,
               BufferStateUpdate.class,
               AllocationStatusUpdate.class,
               ControlSpaceRevoke.class },
      cardinality = Cardinality.SINGLE)
public interface BufferResourceManager extends ResourceManager {
}
```

The `BufferResourceManager` interface, as shows the code snippet below, defines the default ports this resource manager can connect with. A buffer resource should accept BufferAllocations and AllocationRevoke messages. It can send BufferRegistration and BufferSystemDescription which specify the existence and behavior of the resource manager. The BufferStateUpdate and AlloactionStateUpdate will inform the energy app of the changed behavior of this resource manager. The ControlSpaceRevoke message to conclude withdraws a previous BufferStateUpdate, which is a ControlSpaceUpdate. For more information on these messages and the relation between a resource manager and an energy app, see the “Generic Explanation Energy Flexibility Interface”  document.

```java
    private static final Unit<Energy> WH = (Unit<Energy>) SI.WATT.times(NonSI.HOUR);
    private static final int BATTERY_ACTUATOR_ID = 1;

    private Config configuration;
    private TimeService timeService;
    private ScheduledExecutorService scheduler;
    private BatteryState currentBatteryState;
    private Date changedStateTimestamp;
    private Actuator batteryActuator;
    private BufferRegistration<Energy> batteryBufferRegistration;
```

The snippet above shows the private members of the `BatteryManager` class. The unit WH is not defined in the SI or NonSI package, therefore it is defined here.

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

      @Reference(optional = false)
      public void setScheduledExecutorService(ScheduledExecutorService scheduler) {
          this.scheduler = scheduler;
      }
```

The `TimeService` and the `ScheduledExecutorService` is set in the same way as seen in the `BatterySimulation` section.

```java
    @Activate
    public void activate(BundleContext bundleContext, Map<String, Object> properties)
    {
        // create a configuration
        configuration = Configurable.createConfigurable(Config.class, properties);
        log.debug("BatteryManager Activated");
    }

    @Deactivate
    public void deactivate() {
    }
```

The `activate` method is annoted with `@Activate` and thus will be called after activation. Its only function is to retreive the `Config` information. The `deactivate` is empty.

Next override the method `startRegistration`, it is called when the bundle itself is up and running. Its task is to set up the BufferRegistration, the `BufferSystemDescription` and the `BufferStatsUpdate` message, in order to inform the energy app of its existence.

The behavior of the battery  we want to describe is:

|           | Charging          | Charging  | Idle              | Idle      | Discharging       | Discharging |
|-----------|-------------------|-----------|-------------------|-----------|-------------------|-------------|
| Range(x)  | Charge speed(x/s) | Power (W) | Charge speed(x/s) | Power (W) | Charge speed(x/s) | Power (W)   |
| 0-5000    | 0.3968            | 1460      | 0                 | 0         | -0.3968           | -1400       |
| 5000-6000 | 0.2778            | 1050      | 0                 | 0         | -0.3968           | -1400       |

The leakage of the battery will be 0.0001x/s

```java
    @Override
    protected List<? extends ResourceMessage> startRegistration(BatteryState batteryState) {
        // safe current state of the battery
        currentBatteryState = batteryState;
        changedStateTimestamp = timeService.getTime();

        // ---- Buffer registration ----
        // create a battery actuator for electricity
        batteryActuator = new Actuator(BATTERY_ACTUATOR_ID,
                                       "Battery",
                                       CommoditySet.onlyElectricity);
        // create a buffer registration message
        Set<Actuator> actuators = new HashSet<Actuator>();
        actuators.add(batteryActuator);
        batteryBufferRegistration = new BufferRegistration<Energy>(
											configuration.resourceId(),
     										changedStateTimestamp,
                                            toSeconds(0),
                                            "Battery level",
                                            WH,
                                            actuators);
```

For the `BufferRegistration`, a list of actuators is needed. In the case of the `BatteryManager` it is only one actuator, specifying the behavior of the simulated battery. The battery uses only the commodity electricity. The buffer will be directly available, therefore zero allocation delay. It fill level unit is WH.

For the `BufferSystemDescription` behavior of the actuators must be described and the leakage function of the buffer.

```java
        // ---- Buffer system description ----
        // create a behavior of the battery
        ActuatorBehaviour batteryActuatorBehaviour =
			makeBatteryActuatorBehaviour(batteryActuator.getActuatorId());
        // create the leakage function of the battery
        FillLevelFunction<LeakageRate> bufferLeakageFunction =
			FillLevelFunction.<LeakageRate> create(0)
             		    	 .add(6000, new LeakageRate(0.0001))
    						 .build();

        // create the buffer system description message
        Set<ActuatorBehaviour> actuatorsBehaviours =
			new HashSet<ActuatorBehaviour>();
        actuatorsBehaviours.add(batteryActuatorBehaviour);
        BufferSystemDescription sysDescr =
			new BufferSystemDescription(batteryBufferRegistration,
         								changedStateTimestamp,
   										changedStateTimestamp,
   										actuatorsBehaviours,
   										bufferLeakageFunction);
```

For the actuator behavior a helper method is created, which will be explained shortly. Next the leakage function is defined using the builder functionality of the `FillLevelFunction` class. It has a simple linear leakage function from 0 to 6000 it leaks with 0.0001. Note that a buffer uses a domain less value to indicate its filling level. In this case it is defined as a number between 0 and 6000.

The `BufferSystemDescription` can now be made. It links to the `BufferRegistration` message and is valid from the moment of creation.

The helper method `makeBatteryActuatorBehaviour` defines the behavior as specified in the battery behavior table. To safe space, the definitions for the idle and discharge are omitted. Also some small helper classes are not shown.

First three transitions have to be made, which indicate the transition towards the idle state, towards the charge state and towards the discharge state. With these three transitions, the transition graph can be constructed, which is fully connected. So from all states you can go to all other states. In this example the transitions are made in a helper class which sets the cost to zero and connects no timers.

Next the fill level functions have to be made. For this the `FillLevelFunction` class has a builder construction to make one. In this example all costs are zero.

Now the fill level functions are available, the running modes can be created, which binds a battery mode with a fill level function and a transition, and gives it all a name.

To conclude an `ActuatorBehaviour` object can be made for the given actuator actuatorId, containing all three running modes.

```java
    private ActuatorBehaviour makeBatteryActuatorBehaviour(int actuatorId) {
        // make three transitions holders
        Transition toChargingTransition = makeTransition(0);
        Transition toIdleTransition = makeTransition(1);
        Transition toDisChargingTransition = makeTransition(2);

        // create the transition graph, it is fully connected in this case
        Set<Transition> chargeTransition = new HashSet<Transition>();
        chargeTransition.add(toIdleTransition);
        chargeTransition.add(toDisChargingTransition);

	 --- additional transitions here ---

        // make the fill level functions
		FillLevelFunction<RunningModeBehaviour> chargeFillLevelFunctions =
    		FillLevelFunction.<RunningModeBehaviour> create(0)
          		.add(5000, new RunningModeBehaviour(0.3968,
  								CommodityMeasurables.create()
                      								.electricity(toWatt(1460))
   													.build(),
 			         			toEuroPerHour(0)))
    			.add(6000, new RunningModeBehaviour(0.2778,
   								CommodityMeasurables.create()
    												.electricity(toWatt(1050))
    												.build(),
   								toEuroPerHour(0)))
    			.build();

	 --- additional fill level functions here ---

        // Based on the fill level functions and the transitions
        RunningMode<FillLevelFunction<RunningModeBehaviour>> chargeRunningMode =
			new RunningMode<FillLevelFunction<RunningModeBehaviour>>(
					BatteryMode.CHARGE.ordinal(),
       				"charging",
					chargeFillLevelFunctions,
					chargeTransition);

	 --- additional running modes here ---

        // return the actuator behavior with the three running modes
        return ActuatorBehaviour.create(actuatorId)
                                .add(idleRunningMode)
                                .add(chargeRunningMode)
                                .add(dischargeRunningMode)
                                .build();
    }
```

Now going back to the `startRegistration` method, the last message is the `BufferStateUpdate`. This message informs the energy app of the latest state, in this case the initial state.

```java
        // ---- Buffer state update ----
        // create running mode
        Set<ActuatorUpdate> currentRunningMode =
			makeBatteryRunningModes(batteryActuator.getActuatorId(),
  				           			batteryState.getCurrentMode());

        // create buffer state update message
        Measure<Double, Energy> currentFillLevel = getCurrentFillLevel(batteryState);
        BufferStateUpdate<Energy> update =
			new BufferStateUpdate<Energy>(batteryBufferRegistration,
          								  changedStateTimestamp,
    									  changedStateTimestamp,
    									  currentFillLevel,
    									  currentRunningMode);

        log.debug("Battery manager start registration completed.");
        // return the three messages
        return Arrays.asList(batteryBufferRegistration, sysDescr, update);
    }
```

To create a `BufferStateUpdate` message, a current fill level and a current running mode is needed. The Running mode is created using a helper method which creates an `ActuatorUpdate` message for the battery actuator. Another helper method is used to calculate the current fill level.

```java
    /**
     * Create the battery actuator set, based on the battery mode
     *
     * @param batteryMode
     * @return
     */
    private Set<ActuatorUpdate> makeBatteryRunningModes(int actuatorId,
    BatteryMode batteryMode) {
        Set<ActuatorUpdate> runningModes = new HashSet<ActuatorUpdate>();
        ActuatorUpdate actuatorUpdate =
			new ActuatorUpdate(actuatorId, batteryMode.ordinal(), null);
        runningModes.add(actuatorUpdate);
        return runningModes;
    }
```

The fill level is calculated out of the current state of charge of the battery multiplied by the total capacity of the battery.

```java
    /**
     * current fill level is calculated by multiplying the total capacity
     * with the state of charge.
     *
     * @param batteryState
     * @return
     */
    private Measure<Double, Energy> getCurrentFillLevel(BatteryState batteryState) {
        double stateOfCharge = batteryState.getStateOfCharge();
        Measurable<Energy> totalCapacity = batteryState.getTotalCapacity();

        return Measure.valueOf(stateOfCharge * totalCapacity.doubleValue(WH), WH);
    }
```

At the end of the `startRegistration` method all three messages: `BufferRegistration`, `BufferSystemDescription` and `BufferStatsUpdate` are returned as a List.

Now override the method `updateState`, which will be called automatically when the battery changes its state.

The first check is to see if the current state has changed, if not do nothing. Otherwise a `BufferStateUpdate` message is created in the same manner as in the `startRegistration` method as described before, but now based on the passed `batteryState` of course.

```java
    @Override
    protected List<? extends ResourceMessage> updatedState(BatteryState batteryState)
    {
        log.debug("Receive battery state update, mode=" + batteryState.getCurrentMode());

        // Do nothing if there is no useful battery state change request
        if (batteryState.equals(currentBatteryState)) {
            return Collections.emptyList();
        } else { // battery state change request
            // save previous state
            currentBatteryState = batteryState;
            changedStateTimestamp = timeService.getTime();

            // this state change is immediately valid
            Date validFrom = changedStateTimestamp;

            // create running mode
            Set<ActuatorUpdate> currentRunningMode =
  				makeBatteryRunningModes(batteryActuator.getActuatorId(),
       									batteryState.getCurrentMode());
            // create buffer state update message
            Measure<Double, Energy> currentFillLevel =
  				getCurrentFillLevel(batteryState);
            BufferStateUpdate<Energy> update =
  				new BufferStateUpdate<Energy>(batteryBufferRegistration,
                                              changedStateTimestamp,
											  validFrom,
											  currentFillLevel,
											  currentRunningMode);
            // return state update message
            return Arrays.asList(update);
        }
    }
```

The last method that needs to be overridden is the `receivedAllocation` method. This method handles a new allocation request from the energy app. The allocation message for a buffer contains an allocation for one or more actuators.

For each actuator a `batteryMode` change must be scheduled. Therefore the batteryMode is determined by the `runningModeId` in the allocation. The delay is being calculated based on the current time and the specified start time. Now both parameters are given to a helper method to set up the schedule. After that an `AllocationStatusUpdate` message is send to the energy app to inform of the acceptance of the allocation request.

```java
    @Override
    protected BatteryControlParameters receivedAllocation(ResourceMessage message) {
        // is the message a buffer allocation?
        if (message instanceof BufferAllocation) {
            BufferAllocation bufferAllocation = (BufferAllocation) message;
            log.debug("Received allocation " + bufferAllocation);

            // process all actuator allocations
            Set<ActuatorAllocation> allocations = bufferAllocation.getActuatorAllocations();
            for (ActuatorAllocation allocation : allocations) {
                // determine battery mode and specified delay.
                BatteryMode batteryMode = BatteryMode.values()[allocation.getRunningModeId()];
                long delay = allocation.getStartTime().getTime() –
							 	timeService.getCurrentTimeMillis();
                delay = (delay <= 0 ? 1 : delay);

                // set up the switch to the battery mode after the specified delay
                scheduleSwitchToBatteryMode(delay, batteryMode);

                // Send to attached energy app the acceptance of the request
                allocationStatusUpdate(
					new AllocationStatusUpdate(timeService.getTime(),
                                               bufferAllocation,
                                               AllocationStatus.ACCEPTED,
                                               ""));
            }

            // all battery mode changes are scheduled at time, so nothing to return
            return null;
        } else {
            log.warn("Unexpected resource (" + message.toString()
                     + ") message type (" + message.getClass().getName()
                     + ")received");
            return null;
        }
    }
```

The helper method to set up a schedule for a new battery mode is given in the code snippet below. It creates an inner `Runnable` class which sends the new battery mode to the battery via the `sendControlParameters` method.
This inner class is now scheduled, using the scheduler service, to start once after the calculated delay.

```java
    private void scheduleSwitchToBatteryMode(final long delay,
      final BatteryMode newBatteryMode) {
        final Runnable allocationHelper = new Runnable() {
            @Override
            public void run() {
                sendControlParameters(new BatteryControlParameters() {
                    @Override
                    public BatteryMode getMode() {
                        return newBatteryMode;
                    }
                });
                log.debug("Has set up allocation at " + delay
                          + "ms for batteryMode=" + newBatteryMode);
            }
        };
        scheduler.schedule(allocationHelper, delay, TimeUnit.MILLISECONDS);
    }
```
