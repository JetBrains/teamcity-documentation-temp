[//]: # (title: teamcity-info.xml)
[//]: # (auxiliary-id: teamcity-info.xml)

As an obsolete approach to collect the build script collect information, you can generate an XML file called `teamcity-info.xml` in the root build directory. When the build finishes, this file will automatically be uploaded as a build artifact and processed by the TeamCity server.

Note that this approach __can be discontinued__ in the future TeamCity versions, so [service messages](service-messages.md) approach is recommended instead. In case service messages do not work for you, let us know the details and describe the case via [email](troubleshooting.md).

## Modifying Build Status

TeamCity has the ability to change the build status directly from the build script. You can set the status (build failure or success) and change the text of the build status (for example, note the number of failed tests if the test framework is not supported by TeamCity).

It is possible to set the following information for the build:

* __Build number__ — sets the new number for the finished build. You can reference the TeamCity-provided build number using `{build.number}`.
* __Build status__ — changes the build status. Supported values are `FAILURE` and `SUCCESS`.
* __Status text__ — modifies the text of build status. You can replace the TeamCity-provided status text or add a custom part before or after the standard text. Supported `action` values are "append", "prepend", and "replace".

Example of the `teamcity-info.xml` file:

```XML
<build number="1.0.{build.number}">
   <statusInfo status="FAILURE"> <!-- or SUCCESS -->
      <text action="append"> fitnesse: 45</text>
      <text action="append"> coverage: 54%</text>
   </statusInfo>
</build>

```

<note>

It is up to you to figure out how to retrieve test results that are not supported by TeamCity and accurately add them to the `teamcity-info.xml` file.

</note>

## Reporting Custom Statistics

It is possible to provide [custom charts](customizing-statistics-charts.md) in TeamCity. Your build can provide data for such graphs using `teamcity-info.xml` file.

### Storing Data in teamcity-info.xml

This file should be created by the build in the root directory of the build. You can publish multiple statistics (see the details on the data format below) and create separate charts for each set of values.

The `teamcity-info.xml` file is to contain the code in the following format (you can combine various data in the `teamcity-info.xml` file):


```XML
<build>
   <statisticValue key="chart1Key" value="342"/>
   <statisticValue key="chart2Key" value="53"/>
</build>

```

The `key` must not be equal to any of [predefined keys](customizing-statistics-charts.md#Modifying+Predefined+Project-level+Charts). The `value` must be a positive/negative integer of up to 13 digits. Float values with up to 6 decimal places are supported.

The key here relates to the key of the __valueType__ tag used when describing the chart.

### Describing custom charts

See [Customizing Statistics Charts](customizing-statistics-charts.md) page for the detailed description.
