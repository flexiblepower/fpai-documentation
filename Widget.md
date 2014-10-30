# Developing a Widget

Each resource driver can have a widget, a web interface component. The `BatterySimulation` has one to visualize the state of charge (`soc`), the total capacity of the simulated battery (`totalCapacity`) and the battery mode (mode) it is in. A widget consists out of a java class and three resources: a html file, a java script file and a style sheet.

The skeleton project has default implementations of the resources, which are stored in the `/res/widgets/ExampleWidget` directory. First rename the `ExampleWidget` directory into `BatteryWidget`.

The code snippet below shows the java `BatteryWidget` class, which  must implement the Widget interface and provide a method to retrieve the status of the battery. In the first part of the code the `Update` class is defined, a holder the information for the widget. Then the `update` method, which creates an update object and fills it with the necessary information. This method will be called from javascript, as will be explained next.

```java
public class BatteryWidget implements Widget {
    public static class Update {
        private final int soc;
        private final String totalCapacity;
        private final String mode;

        public Update(int soc, String totalCapacity, String mode) {
            this.soc = soc;
            this.totalCapacity = totalCapacity;
            this.mode = mode;
        }

        public int getSoc() {
            return soc;
        }

        public String getTotalCapacity() {
            return totalCapacity;
        }

        public String getMode() {
            return mode;
        }
    }

    private final BatterySimulation simulation;

    public BatteryWidget(BatterySimulation simulation) {
        this.simulation = simulation;
    }

    public Update update() {
        BatteryState state = simulation.getCurrentState();
        double soc = state.getStateOfCharge();
        int socPercentage = (int) (soc * 100.0);
        double capacity = state.getTotalCapacity().doubleValue(KWH);
        BatteryMode mode = state.getCurrentMode();
        return new Update(socPercentage, String.format("%2.1f kWh", capacity),
       mode.toString());
    }

    @Override
    public String getTitle(Locale locale) {
        return "Storage Device Manager";
    }
}
```

The widget itself is a html file, which must be called `index.html`. It loads two style sheets, own default widget style sheet and a specific style sheet for this widget. After specifying a number of javascript libraries, the body is defined as a class called “widget”. In the battery simulation 8 pictures are created showing different state of charges of the battery, default the empty one is loaded. After that the 3 labels are defined, which will be update via javascript.

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
	<title>Simulated Battery Panel</title>
</head>
<body class="widget">
	<img id="icon" src="1.png" alt="Battery" />
	<img id="loading" src="/img/loading.gif" alt="Loading..." />

	<p><label>State of charge</label> <span id="soc">...</span></p>
	<p><label>Total capacity</label> <span id="tc">...</span></p>
	<p><label>Current mode</label> <span id="mode">...</span></p>
</body>
</html>
```

The javascript runs when the window is loaded. It creates a new widget which will call every 1000ms the update function on the java widget class. The data the this method produces is serialized and passed as data to the function. This function updates the three labels and adapts the state of charge picture to the new soc of the battery. The html file refers to the hash tags fromt he javascript file used the `id` attribute.

The content of the `script.js` file is as follows:
```javascript
$(window).load(function() {
	w = new widget("update", 1000, function(data) {
		$("#loading").detach();
		$("p").show();
		$("#soc").text(data.soc + "%");
		$("#tc").text(data.totalCapacity);
		$("#mode").text(data.mode);

		if(data.soc > 87) {
			$("#icon").attr("src", "8.png");
		} else if(data.soc > 75) {
			$("#icon").attr("src", "7.png");
		} else if(data.soc > 62) {
			$("#icon").attr("src", "6.png");
		} else if(data.soc > 50) {
			$("#icon").attr("src", "5.png");
		} else if(data.soc > 37) {
			$("#icon").attr("src", "4.png");
		} else if(data.soc > 25) {
			$("#icon").attr("src", "3.png");
		} else if(data.soc > 12) {
			$("#icon").attr("src", "2.png");
		} else {
			$("#icon").attr("src", "1.png");
		}
	});
});
```

The content of the `style.css` file is as follows:
```javascript
#loading {
	position: absolute;
	left: 45%;
	top: 40%;
}

#icon {
	position: absolute;
	bottom: 0;
	right: 1em;
	height: 120px;
}

p {
	height: 1.2em;
	display: none;
}
```