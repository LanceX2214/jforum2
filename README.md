## Check [http://jforum.net](jforum.net) for the currently maintained version

## Kieker Trace - Quick Instructions

This repository is already instrumented with Kieker + AspectJ.
Use the steps below to generate trace graphs (`.pdf`) for jforum.

### 1. Prerequisites

- Java 8 (recommended for running jforum and tooling in this setup)
- `ant` (build jforum WAR)
- `dot` from Graphviz (convert `.dot` to `.pdf`)
- Tomcat (this repo already has `tools/apache-tomcat-8.5.100`)

### 2. Build jforum WAR

```bash
cd jforum2
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk use java 8.0.472-amzn
ant clean dist
```

WAR output:

```text
jforum2/dist/jforum-2.1.8.war
```

### 3. Run jforum with AspectJ javaagent (to generate traces)

Deploy `jforum2/dist/jforum-2.1.8.war` to Tomcat as `jforum.war`, then start Tomcat with:

```bash
-javaagent:/absolute/path/to/jforum2/WEB-INF/lib/aspectjweaver-1.9.7.jar
-Dkieker.monitoring.writer.filesystem.FileWriter.customStoragePath=/absolute/path/to/jforum2/build/kieker-traces/jforum
```

After startup, visit one or more pages (example):

```text
http://127.0.0.1:18080/jforum/install/install.page
http://127.0.0.1:18080/jforum/forums/list.page
```

Trace files will be written under:

```text
jforum2/build/kieker-traces/jforum/<kieker-run-folder>/
```

### 4. Generate DOT and analysis output

Run from repository root (`.../cloud-native/jforum`):

```bash
TRACE_ANALYSIS_BIN="PartsUnlimitedMRP/tools/trace-analysis-2.0.2/bin/trace-analysis"
TRACE_PARENT="jforum2/build/kieker-traces/jforum"
TRACE_INPUT_DIR="$(find "$TRACE_PARENT" -mindepth 1 -maxdepth 1 -type d | sort | tail -n 1)"
OUT_DIR="jforum2/trace-analysis-output/jforum"

mkdir -p "$OUT_DIR"

"$TRACE_ANALYSIS_BIN" \
  --inputdirs "$TRACE_INPUT_DIR" \
  --outputdir "$OUT_DIR" \
  --plot-Assembly-Component-Dependency-Graph responseTimes-ns \
  --plot-Assembly-Operation-Dependency-Graph responseTimes-ns \
  --plot-Deployment-Operation-Dependency-Graph responseTimes-ns \
  --print-Execution-Traces \
  --print-System-Model
```

### 5. Convert DOT to PDF

```bash
OUT_DIR="jforum2/trace-analysis-output/jforum"

dot "$OUT_DIR/assemblyComponentDependencyGraph.dot" -T pdf -o "$OUT_DIR/assemblyComponentDependencyGraph.pdf"
dot "$OUT_DIR/assemblyOperationDependencyGraph.dot" -T pdf -o "$OUT_DIR/assemblyOperationDependencyGraph.pdf"
dot "$OUT_DIR/deploymentOperationDependencyGraph.dot" -T pdf -o "$OUT_DIR/deploymentOperationDependencyGraph.pdf"
```

### 6. Generated files

Main outputs:

- `jforum2/trace-analysis-output/jforum/assemblyComponentDependencyGraph.pdf`
- `jforum2/trace-analysis-output/jforum/assemblyOperationDependencyGraph.pdf`
- `jforum2/trace-analysis-output/jforum/deploymentOperationDependencyGraph.pdf`
- `jforum2/trace-analysis-output/jforum/executionTraces.txt`