# 9. Energy App

The energy app receives the `ControlSpace` from the `WashingMachineManager` and returns an `Allocation`.

* Copy the skeleton project and rename it into: `org.flexiblepower.example.energyapp`.
* Within this newly created project go to the src folder and rename the `org.flexiblepower.example.skeleton` package into: `org.flexiblepower.example.energyapp`.
* Make sure this new package is referred to by the `Private-Package` property in the bnd.bnd file. No packages need to be exported by this bundle, since it offers no services that can be used by other components, therefore the `Export-Package` property can be omitted in this case. The file should look as follows.

```
-buildpath: ${default-buildpath}

Bundle-Version: 1.0.0.${qualifier}
Private-Package: org.flexiblepower.example.energyapp
Service-Component: *
```

Create a new class called `EnergyApp` that implements `ControllerManager`.

```java
@Component(designateFactory = EnergyApp.Config.class,
provide = ControllerManager.class)
public class EnergyApp implements ControllerManager {
	
	@Meta.OCD(name = "Simple Energy App")
	interface Config {
		@Meta.AD(deflt = "washingmachine", description = "Resource identifier")
		String[] resourceIds();

	}

	@SuppressWarnings("unchecked")
	@Override
	public void registerResource(ControllableResource<?> resource) {
		if (
TimeShifterControlSpace.class.isAssignableFrom(resource.getControlSpaceType())) {
			TimeshifterController timeshifterController =
new TimeshifterController();
			timeshifterControl-ler.bind((ControllableResource<TimeShifterControlSpace>) resource);
		}
		
	}

	@Override
	public void unregisterResource(ControllableResource<?> resource) {
		// TODO Auto-generated method stub
		
	}
}
```

This class contains the familiar annotation and inner `Config` interface that are necessary for its configurations

It has a `registerResource` method that gets called when a `ControlSpace` is received from a `ResourceManager` (or `ControllableResource` as it is called her) for the first time. This method first checks whether this is a `TimeShifterControlSpace`. When this is the case it constructs a new `TimeShifterController` class.

Create the `TimeShifterController` class that implements `Controller`.

```java
public class TimeshifterController implements Controller<TimeShifterControlSpace> {
	static Logger logger = LoggerFactory.getLogger(TimeshifterController.class);
	private ControllableResource<TimeShifterControlSpace> controllableResource;
	
	public void bind(ControllableResource<TimeShifterControlSpace>
controllableResource) {
		logger.info("Binding resource manager to TimeshifterController");
		this.controllableResource = controllableResource;
		controllableResource.setController(this);
	}

	@Override
	public void controlSpaceUpdated(
			ControllableResource<? extends TimeShifterControlSpace> resource,
			TimeShifterControlSpace controlSpace) {
		logger.info("Received control space with timestamp" +
controlSpace.getValidFrom());
		
long timeInTheMiddle = (controlSpace.getStartAfter().getTime() +
controlSpace.getStartBefore().getTime())/2;
		Allocation allocation = new Allocation(controlSpace,
new Date(timeInTheMiddle),
controlSpace.getEnergyProfile());
		controllableResource.handleAllocation(allocation);
	}
	
}
```

The `controllableResource` contains the reference to the Resource Manager of the washing machine (which is referred to as a `ControllableResource` here). The link between with the ControllableResource is made in the bind method. The `setController` method is used with this class as the argument to make sure that allocations can be sent.

The `controlSpaceUpdated` method is inherited from the `Controller` and is called when a new `ControlSpace` is received. Upon receiving the `ControlSpace` a start time is determined and an `Allocation` is constructed that is given to the `handleAllocation` method of the `ControllableResource`.

Follow the instructions in 3.5.3 to get the `EnergyApp` component up and running. Of course the instructions should not be taken literally, but must refer to the items that are applicable to the `EnergyApp` component.