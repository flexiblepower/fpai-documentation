# 10. Developing a Widget

Each component that is developed within FPAI can use a widget to provide insight to and receive input from the user. The example below explains how the resource driver for the washing machine can be extended with a widget.

* Go to the `org.flexiblepower.example.timeshifter.washingmachine.driver.impl` project.
* Create a `WashingMachineWidget` class in the `org.flexiblepower.example.timeshifter.washingmachine.driver.impl` package.

```java
public class WashingMachineWidget implements Widget {
	public static class Update {
		private final String earliestStartTime;
		private final String latestStartTime;
		private final String programName;
		
		public Update(String earliestStartTime, String latestStartTime, String programName) {
			this.earliestStartTime = earliestStartTime;
			this.latestStartTime = latestStartTime;
			this.programName = programName;
		}

		public String getEarliestStartTime() {
			return earliestStartTime;
		}

		public String getLatestStartTime() {
			return latestStartTime;
		}

		public String getProgramName() {
			return programName;
		}
	}
	
	private final static DateFormat FORMATTER = new SimpleDateFormat("HH:mm:ss");
	private final WashingMachineDriverImpl washingMachineDriverImpl;
	
	public WashingMachineWidget(WashingMachineDriverImpl washingMachineDriverImpl) {
		this.washingMachineDriverImpl = washingMachineDriverImpl;
	}
	
	public Update update() {
		WashingMachineState state = washingMachineDriverImpl.getCurrentState();
		
		return new Update(FORMATTER.format(state.getEarliestStartTime()), FOR-MATTER.format(state.getLatestStartTime()), state.getProgramName());
	}

	@Override
	public String getTitle(Locale locale) {
		return "Washing Machine panel";
	}

}
```

The important feature of this class is the inner `Update` class. This class will be serialized using JSON. This way information is made available to the widget in the GUI of FPAI.

There is a private member variable that holds a reference to the `WashingMachineDriverImpl` object and that is initialized in the constructor.

The update method gets called regularly. In this case it first gets the current state from the `WashingMachineDriverImpl` object and constructs a new Update object to return.

The `getTitle` method returns the title of the widget panel.

Create a `widgets` folder and make a subfolder that has the exact same name as the `WashingMachineWidget` class. This subfolder should contain three files: `script.js`, `style.css` and `index.html`. Additional files such as images that are used by this widget can also be placed here.

This the content of the `script.js` file.

```javascript
$(window).load(function() {
	w = new widget("update", 1000, function(data) {
		$("#loading").detach();
		$("p").show();
		$(".error").hide();
		$("#earliestStartTime").text(data.earliestStartTime);
		$("#latestStartTime").text(data.latestStartTime);
		$("#programName").text(data.programName);
	});
});
```

The JSON object is read and its values are available through the hash tags.

The content of the `index.html` file is as follows:

```html
<!doctype html>
<html>
<head>
	<link rel="stylesheet" href="/css/widget.css" />
	<link rel="stylesheet" href="style.css" />
	<script type="text/javascript" src="/js/vendor/jquery.min.js"></script>
	<script type="text/javascript" src="/js/widget.js"></script>
	<script type="text/javascript" src="script.js"></script>
	<title>Washing Machine Panel</title>
</head>
<body class="widget">
	<img id="icon" src="ariston-washing-machine.png" alt="Washing Machine Panel" />
	<img id="loading" src="/img/loading.gif" alt="Loading..." />

	<p class="error"></p>
	<p><label>Program</label> <span id="programName"></span></p>
	<p><label>Start after</label> <span id="earliestStartTime"></span></p>
	<p><label>Start before</label> <span id="latestStartTime"></span></p>
</body>
</html>
```

The html file refers to the hash tags from the javascript file using the `id` attribute.

The `style.css` file:

```css
#loading {
	position: absolute;
	left: 45%;
	top: 40%;
}

#icon {
	position: absolute;
	bottom: 0;
	right: 1em;
}

p {
	height: 1.2em;
	display: none;
}
```

This file contains the stylesheet for this widget.

The widget is all set up, but it still needs to be started from the `WashingMachineDriverImpl` class. The following code snippets show the changes that had to be made to this class to successfully launch the widget.

```java
/** Reference to the registration as widget */
private ServiceRegistration<Widget> widgetRegistration;
/** Reference to the widget itself */
private WashingMachineWidget widget;
```

First a variable of the `ServiceRegistration` type parameterized with a `Widget` interface is declared, together with a variable of the `WashingMachineWidget` type.

```java
	/**
	 * This method gets called after this component gets a configuration and
	 * after the methods with the Reference annotation are called
	 * 
	 * @param bundleContext
	 *            OSGi BundleContext-object
	 * @param properties
	 *            Map containing the configuration of this component
	 */
	@Activate
	public void activate(BundleContext bundleContext,
			Map<String, Object> properties) {
		// Get the configuration
		config = Configurable.createConfigurable(Config.class, properties);

		// Register us as an ObservationProvider
		String resourceId = config.resourceId();
		observationProviderRegistration = new ObservationProviderRegistra-tionHelper(
				this).observationType(WashingMachineState.class)
				.observationOf(resourceId).register();

		widget = new WashingMachineWidget(this);
		widgetRegistration = bundleContext.registerService(Widget.class, widget, null);
		
		schedulerService.scheduleAtFixedRate(this, 0, 5, ja-va.util.concurrent.TimeUnit.SECONDS);
	}
```

In the activate method the `WashingMachineWidget` is constructed and registered with the `widgetRegistration`. When the `WashingMachineDriverImpl` gets started it will now also launch the `WashingMachineWidget`.

```java
	/**
	 * This method is called before the driver gets destroyed
	 */
	@Deactivate
	public void deactivate() {
		widgetRegistration.unregister();
		scheduledFuture.cancel(false);
		observationProviderRegistration.unregister();
	}
```

Finally, a line of code has to be added to the `deactivate` method to unregister the widget.

