

# Benchmark plug-in

This plug-in is designed for the CI/CD platform, [**Jenkins**](https://jenkins.io/).   
This plug-in adds a post-build capability.  
This plug-in purpose is to benchmark over all of a job builds, boolean and numerical results through tables and graphs.  
This plug-in has also a mechanism of thresholds over numerical values to control a build success or failure.  

For the plug-in one slide features, click [HERE](./doc/170821_Benchmark_Plugin_OneSlide.pdf).

This plug-in is also designed to load any type of JSON or XML result file format.  
This plug-in works best in combination with an automated notification plug-in.  
This plug-in works also as a result viewer for the ['jUnit Plug-in'](https://wiki.jenkins.io/display/JENKINS/JUnit+Plugin) and the ['xUnit Plug-in'](https://wiki.jenkins.io/display/JENKINS/xUnit+Plugin).  
This plug-in is accessible as a Jenkins pipeline component.  

If you are interested  in why this plug-in was created, please find the original proposal [HERE](./doc/170515_Benchmark_Plugin_Proposal.pdf).

## Author
Daniel Mercier

## License
This work is under MIT license (see "license" document).

## Contributing
Please review the [contributing guidelines](./CONTRIBUTING.md) before working on code changes.

## Links
- Click [HERE](./doc/HOW_TO_USE_THE_PLUGIN.md) to access the manual on how to use the plugin,
- Click [HERE](./doc/BUILD_CUSTOM_SCHEMA.md) to access instructions on how to create a Custom Schema,  
- Click [HERE](/src/main/resources/schemas) to access the currently registered schema(s).

## Description
Numerical solvers among other solutions involve complex computations and generate massive amount of numerical values. To ensure their resilience to evolution, they require thorough testing. These tests usually output a range of information from single Boolean success/failure to complex set of numerical properties. Monitoring changes of these numerical properties is critical. Jenkins provides a good set of plugins for Boolean test report but does not have a plug-in to address numerical test reports and the bench-marking of these test results. This Jenkins plug-in addresses this result bench-marking in the form of a Jenkins post-build event combined with a set of tables and graphs.

The inclusion inside Jenkins of this plug-in capabilities starts inside the Job configuration page as a post-build events under the name: **"Benchmark results"**. The plug-in section requires:
- Location of the test result file inside the build workspace,  
- Number of past builds to display.  

Optionally, the section offers a way to set thresholds for specific tests.

During the execution of the post-build event, the plug-in will:  
- Retrieve the specified result file inside the build workspace,  
- Parse it according to either the default or the specified schema (advanced option),  
- Condense the parsed content with the information from the job configuration file,  
- Check set thresholds and fail the build if any were crossed,  
- Save the condensed content in the build area of the master node.  
- Report execution in the build log,  

After each completed build, in the Job initial window, a new icon will appear named **"Benchmark tables"** with access to a new window with two tables:
- First table with the test results for the number of specified builds,  
- Second table with condensed results: average, minimum, maximum, standard deviation.  

Clicking on any of the table lines will open a new window with detailed information about the selected result as well as a graph for numerical results.

**Note:** If the field for the location of the input files is left blank, the plug-in will try to load any file found inside the build history generated by either the 'JUnit Plug-in' or the 'xUnit Plug-in'. This way, the plug-in can be used as an additional form of result display & exploration tool.

## Languages

**English**, **French**  

Contributors are welcome !!!  
Supports left to right as well as right to left.  

The strings to translate are located in the "*. properties" resource files inside:  
```
D:\Git\BenchmarkPlugin\src\main\resources\org\jenkinsci\plugins\benchmark
```  
Steps:  
- Create new files with different locale (currently **'\*_fr'** stands for French, default is English),  
- Copy content and translate the right hand side of the equation,  
- Test the translation using the [Jenkins locale plugin](https://wiki.jenkins.io/display/JENKINS/Locale+Plugin),  
- Check display spacing.  

## Development IDE
IDEA IntelliJ - Maven project 

**Configuration to debug (with defined JENKINS_HOME):**  
```
    hpi:run -Djetty.port=8090 -Dhpi.prefix=/jenkins  
```  

**Configuration to test:**  
```
    test -DforkMode=never -Dsurefire.useFile=false  
```  

**Configuration to generate additional sources (localizer):**  
```
    compile
```
With the localized generated classes inside **target/generated/sources/localizer**, to clear the editor of localization errors, add the folder as a source folder (e.g. 'Files'/'Project Structure'/'Modules').  

**Important:** The Kohsuke stapler plug-in for IDEA IntelliJ is strongly suggested to assist with i18n.  

**Configuration to build the final package:**  
```
    install -Dfindbugs.skip=true  
```  
**Note:** While FindBugs is powerful and useful tool, some standard Java practices are detected as warnings that fails the build without the skip above. If ran without skip, run `mvn findbugs:gui` after to analyze FindBugs findings through the University of Maryland GUI.  
## Registered schemas

To get details about the registered schema(s), please access the content [HERE](/src/main/resources/schemas) or at:  
``` 
    src/main/resources/schemas
```

## Warnings
1. If the schema is valid but the result file does not match the schema. There is a risk for the plug-in to skip relevant data,  
2. This plugin is very dependent on RAM memory.  

**Note:** Jenkins uses somewhere between ~200Mb & ~400Mb for the Master node. Every user that accesses the Jenkins server triggers the loading of all job configurations in memory. It usually does not add too much but this is dependent on the number of jobs.  

With the benchmark plugin:  
- During the post-build event, all the build results are aggregated and combined with the latest results and therefore loaded in memory on the master node,  
- During result exploration, all the build results are also aggregated and loaded in memory for fast operations. The memory is released once the system detects that the user is not exploring results anymore.  
  
An inherent risk exists to overload the memory during processing due to:
- The result files having too many results or related content e.g. stack trace can significantly add to the size,
- Multiple sizable result files being processed at the same time by multiple jobs,

Another inherent risk exists to overload the memory during benchmark display due to:
- Large number of saved builds to a job,
- Large number of results or large related content,
- Large number of users accessing the benchmark plugin display simultaneously.

**Suggestion:** Activate the configuration to keep a fixed number of builds,
**Suggestion:** Use the plug-in option to truncate strings to reduce string content when dealing with long stack traces.

## Various notes on architecture

- Jenkins keeps in RAM memory all the information about every single job. When the Benchmark Plugin loads the results from files to RAM memory, this memory is kept active until deallocated. The size depends on the number of results, the number of builds and the number of jobs with the Benchmark Plugin active. Jenkins does not track when a user stop accessing specific pages so the plugin manages memory persistence with a mechanism of two triggers:
  - A timer located in the back-end that automatically deallocates memory after a fixed time,
  - A ping sent by the client that resets the timer every 30s; and so, as long the user is browsing any of the Benchmark plugin pages.

## Copyrights

The core libraries comes from Jenkins which is under MIT license.

This plugin uses the following additional components:  

| Name       | Reference                                                | License                                              |  
|------------|----------------------------------------------------------|------------------------------------------------------|  
| Gson       | github.com/google/gson                                   | Apache license, version 2.0                          |  
| Chartjs    | chartjs.org                                              | MIT license                                          |  
| JQuery     | jquery.org                                               | MIT license                                          |  
| Datatables | datatables.net                                           | MIT license                                          |  
| FileSaver  | github.com/eligrey/FileSaver.js                          | MIT license                                          |  
| TextToHTML | http://www.java2s.com/Code/Java/Servlets/TextToHTML.htm  | LGPL                                                 |  

(Please find the library license documents in the **legal** folder)  

## Useful references

Chart.js examples - http://tobiasahlin.com/blog/chartjs-charts-to-get-you-started/  
Setting Jenkins API -https://wiki.jenkins.io/display/jenkins/remote+access+api  

## TODO

- Add more languages (See above for i18n),  
- Add unit tests,  
- Improve performances,  
- Add registered schema(s),  

- Add graph with variations of passed/failed for the series of builds,  
- Create **Comparator** for the user to select one or more results in order to compare them through graphs and tables,  
