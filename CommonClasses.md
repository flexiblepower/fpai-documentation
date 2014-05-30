# 4. Common classes and interfaces

## Measurables
In the FPAI there are several interfaces working with physical quantities. For all of these quantities there are multiple units you could use: seconds, minutes, hours, joules, kilowatt hours... One programmer might prefer working with joules while another might prefer working with kilowatt hours. In order to prevent confusion and cumbersome manual conversion we have adopted the `javax.measure` packages from the [JScience](http://jscience.org) project. These packages provide classes and interfaces which combine numbers and units in a *measurable* object. These objects are so commonly used in the FPAI that they are part of the `flexiblepower.api` bundle.

The package provides a set of [SI-units](http://en.wikipedia.org/wiki/International_System_of_Units) (such as second and joule) and NonSI-units (such as hour and Kilowatt hour). Measurables are typed with the generic `Measurable` interface.

```java
// Static import makes working with units easier.
import static javax.measure.unit.SI.*;
import static javax.measure.unit.NonSI.*;

// Create a measureable which stores an amount of power
Measurable<Power> power = Measure.valueOf(50, WATT);

// Create a measureable which stores a duration
Measurable<Duration> hour = Measure.valueOf(1, HOUR);

// Compare two measureables. This should result in true.
boolean res = Measure.valueOf(1000, WATT).equals(Measure.valueOf(1, KILO(WATT)))

// Compare two measureables. This sholud result in a value < 0
int compare = Measure.valueOf(1, JOULE).compareTo(Measure.valueOf(1, KWH));

// Get the value of a measurable as a double. This should result in 3600.0.
double seconds = Measure.valueOf(1, HOUR).doubleValue(SECOND);
```

## TimeUtil

The Java API provides a `Date` class to represent a time. Since we work a lot with Measurable<Duration> in the FPAI, there is a utility class called `TimeUtil` which provides some methods for combining the two.

```java
Date now = new Date();

Date oneHourInTheFuture = TimeUtil.add(now, Measure.valueOf(1, HOUR));
Date oneHourInThePast = TimeUtil.subtract(now, Measure.valueOf(1, HOUR));
Measureable<Duration> twoHours = TimeUtil.difference(oneHourInThePast, oneHourInTheFuture);
```

**Note:** as you will read in the TimeSource section we don't recommand using `new Date()` in the FPAI. This is just for demonstration purposes.

## ConstraintList

## EnergyProfile

## TimeSource

## ScheduledExecutionService