# helm

## Charts

Helm uses a packaging format called charts. A chart is a collection of files that describe a related set of Kubernetes resources. A single chart might be used to deploy something simple, like a memcached pod, or something complex, like a full web app stack with HTTP servers, databases, caches, and so on.

Charts are created as files laid out in a particular directory tree, then they can be packaged into versioned archives to be deployed.

### Charts directory structure

A chart is organized as a collection of files inside of a directory. The directory name is the name of the chart (without versioning information). 

Inside of this directory, Helm will expect a structure that matches this:


```
sample/
  Chart.yaml          # A YAML file containing information about the chart
  values.yaml         # The default configuration values for this chart
  charts/             # A directory containing any charts upon which this chart depends.
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  requirements.yaml   # OPTIONAL: A YAML file listing dependencies for the chart  

```

### Charts and Versioning

Every chart must have a version number. A version must follow the SemVer 2 standard. Packages in repositories are identified by name plus version.

For example, an nginx chart whose version field is set to version: 1.2.3 will be named:

```
nginx-1.2.3.tgz
```

### The appVersion field

appVersion field is not related to the version field. It is a way of specifying the version of the application. For example, the drupal chart may have an appVersion: 8.2.1, indicating that the version of Drupal included in the chart (by default) is 8.2.1. This field is informational, and has no impact on chart version calculations.

### Deprecating Charts

When managing charts in a Chart Repository, it is sometimes necessary to deprecate a chart. The optional deprecated field in Chart.yaml can be used to mark a chart as deprecated. If the latest version of a chart in the repository is marked as deprecated, then the chart as a whole is considered to be deprecated. 

### CHART LICENSE

A LICENSE is a plain text file containing the license for the chart. The chart can contain a license as it may have programming logic in the templates and would therefore not be configuration only. There can also be separate license(s) for the application installed by the chart, if required.

## README

A README for a chart should be formatted in Markdown (README.md), and should generally contain:

* A description of the application or service the chart provides
* Any prerequisites or requirements to run the chart
* Descriptions of options in values.yaml and default values
* Any other information that may be relevant to the installation or configuration of the chart

### NOTES

The chart can also contain a short plain text templates/NOTES.txt file that will be printed out after installation, and when viewing the status of a release. This file is evaluated as a template, and can be used to display usage notes, next steps, or any other information relevant to a release of the chart. For example, instructions could be provided for connecting to a database, or accessing a web UI. Since this file is printed to STDOUT when running helm install or helm status, it is recommended to keep the content brief and point to the README for greater detail.

## Chart dependecies

In Helm, one chart may depend on any number of other charts. These dependencies can be dynamically linked through the requirements.yaml file or brought in to the charts/ directory and managed manually.

### Managing Dependencies with requirements.yaml

A requirements.yaml file is a simple file for listing your dependencies.

```
# parentchart/requirements.yaml
dependencies:
  - name: subchart1
    version: 1.2.3
    repository: http://example.com/charts
    alias: subchart1-apache # optional
    condition: subchart1.enabled,global.subchart1.enabled
    tags:
      - front-end
      - subchart1    
  - name: subchart2
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart2.enabled,global.subchart2.enabled
    tags:
      - back-end
      - subchart2
```

* name: The name field is the name of the chart you want.
* version: The version field is the version of the chart you want.
* repository: The repository field is the full URL to the chart repository. Note that you must also use helm repo add to add that repo locally.
* alias: Adding an alias for a dependency chart would put a chart in dependencies using alias as name of new dependency.We can use alias in cases where they need to access a chart with other name(s).
* condition: The condition field holds one or more YAML paths (delimited by commas). If this path exists in the top parent’s values and resolves to a boolean value, the chart will be enabled or disabled based on that boolean value. Only the first valid path found in the list is evaluated and if no paths exist then the condition has no effect.
* tags: The tags field is a YAML list of labels to associate with this chart. In the top parent’s values, all charts with tags can be enabled or disabled by specifying the tag and a boolean value.    

```
# parentchart/values.yaml

subchart1:
  enabled: true
tags:
  front-end: false
  back-end: true

```  

Managing charts with requirements.yaml is a good way to easily keep charts updated, and also share requirements information throughout a team.

**Tags and Condition Resolution**

* Conditions (when set in values) always override tags.
* The first condition path that exists wins and subsequent ones for that chart are ignored.
* Tags are evaluated as ‘if any of the chart’s tags are true then enable the chart’.
* Tags and conditions values must be set in the top parent’s values.
* The tags: key in values must be a top level key. Globals and nested tags: tables are not currently supported.

