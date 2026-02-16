# jforum2 + Kieker Instrumentation

This project now includes Kieker monitoring via AspectJ load-time weaving.

## Changes Applied

- Added runtime jars to `WEB-INF/lib/`:
  - `kieker-1.14.jar`
  - `aspectjrt-1.9.7.jar`
  - `aspectjweaver-1.9.7.jar`
- Added weaving config: `src/META-INF/aop.xml`
- Added monitoring config: `src/META-INF/kieker.monitoring.properties`
- Updated `build.xml` to copy `src/META-INF/**` into `WEB-INF/classes`
- Added graceful flush in `src/net/jforum/JForum.java` (`destroy()` calls `MonitoringController.getInstance().terminateMonitoring()`).

## Build

Use Java 8 and Ant:

```bash
cd jforum2
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk use java 8.0.472-amzn
ant clean dist
```

WAR output:

- `dist/jforum-2.1.8.war`

## Run with instrumentation

Deploy the WAR to Tomcat/Jetty and start container JVM with:

```bash
-javaagent:/absolute/path/to/aspectjweaver-1.9.7.jar
```

Recommended extra JVM option (write traces to a fixed folder):

```bash
-Dkieker.monitoring.writer.filesystem.FileWriter.customStoragePath=/absolute/path/to/jforum-traces
```

Note: target trace directory must exist before startup.

## Trace Analysis

After generating traffic, run trace analysis:

```bash
trace-analysis-1.14/bin/trace-analysis \
  --inputdirs /absolute/path/to/jforum-traces \
  --outputdir trace-analysis-output/jforum \
  --plot-Assembly-Component-Dependency-Graph responseTimes-ns \
  --plot-Assembly-Operation-Dependency-Graph responseTimes-ns \
  --plot-Deployment-Operation-Dependency-Graph responseTimes-ns \
  --print-Execution-Traces \
  --print-System-Model
```

Convert `.dot` to PDF (Graphviz):

```bash
dot trace-analysis-output/jforum/assemblyComponentDependencyGraph.dot -T pdf -o trace-analysis-output/jforum/assemblyComponentDependencyGraph.pdf
dot trace-analysis-output/jforum/assemblyOperationDependencyGraph.dot -T pdf -o trace-analysis-output/jforum/assemblyOperationDependencyGraph.pdf
dot trace-analysis-output/jforum/deploymentOperationDependencyGraph.dot -T pdf -o trace-analysis-output/jforum/deploymentOperationDependencyGraph.pdf
```
