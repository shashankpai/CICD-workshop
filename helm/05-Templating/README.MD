# Helm Templates

We will learn in this section 
- Helm template basics
- Template values
- Built-in objects
- Helm template functions
- Helm template control structures
- Generating release notes
- Helm template variables
- Helm template validation
- Enabling code reuse with named templates and library charts
- Updating and deploying the Guestbook chart



### Helm Template Basics

Helm templates dynamically generate Kubernetes YAML (or JSON) resources using a mix of default and user-provided values. In **05-Templating**, the `helm create` command generates starter templates, found at `05-Templating/guestbook/templates/`. Here's an example from `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "guestbook.fullname" . }}
  labels: {{- include "guestbook.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels: {{- include "guestbook.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations: {{- toYaml . | nindent 8 }}
      {{- end }}
      labels: {{- include "guestbook.selectorLabels" . | nindent 8 }}
```

### Understanding Helm Templates

Helm templates use Go templates, enhanced with Helm-specific features. Actions, enclosed in `{{ }}`, are used for data processing or control structures like conditionals and loops. Actions are processed and removed during Helm operations, resulting in valid Kubernetes YAML.

### Template Values

Values, configured in Helm charts, are integrated into templates to generate Kubernetes resources dynamically. For instance, consider the `configmap.yaml` template in `05-Templating/examples/values-example/templates/`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: values-example
data:
  config.properties: |-
    chapterNumber={{ .Values.chapterNumber }}
    chapterName={{ .Values.chapterName }}
```

With default values from `values.yaml`:

```yaml
chapterNumber: 6
chapterName: Understanding Helm Templates
```

This renders as:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: values-example
data:
  config.properties: |-
    chapterNumber=6
    chapterName=Understanding Helm Templates
```


You can use the `helm template` command to render templates locally without installing them to a cluster:

```sh
helm template example 05-Templating/examples/values-example
```

This renders the ConfigMap with actual values. `.Values` is a key built-in object for accessing chart values. Other built-in objects will be explored further in this chapter.

### Built-in Objects

Built-in objects are crucial for creating Helm charts, offering access to various metadata and features. Here’s an overview of the main built-in objects in Helm:

| Object         | Definition                                                         |
|----------------|--------------------------------------------------------------------|
| `.Values`      | Access values from `values.yaml` or provided via flags.            |
| `.Release`     | Retrieve metadata about the Helm release (e.g., name, namespace).  |
| `.Chart`       | Access metadata from the `Chart.yaml` file (e.g., chart name, version). |
| `.Template`    | Metadata about the current template (e.g., file path).             |
| `.Capabilities`| Information about the Kubernetes cluster (e.g., API versions, Kubernetes version). |
| `.Files`       | Access files within the Helm chart directory.                       |

#### Example Usage

**Values File** (`05-Templating/examples/values-example/values.yaml`):
```yaml
books:
  harryPotter:
    - The Sorcerer's Stone
    - The Chamber of Secrets
    - The Prisoner of Azkaban
  lotr:
    - The Fellowship of the Ring
    - The Two Towers
    - Return of the King
```

**Accessing Values**:
```yaml
{{ .Values.books.harryPotter }}
{{ .Values.books.lotr }}
```

#### The `.Release` Object

Used to retrieve metadata about the Helm release:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
data:
  config.properties: |-
    namespace={{ .Release.Namespace }}
```

**Example Output**:
```yaml
$ helm template release-example 05-Templating/examples/release-example
---
# Source: release-example/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: release-example
data:
  config.properties: |-
    namespace=default
```

**Attributes**:
- `.Release.Name`: Release name
- `.Release.Namespace`: Namespace
- `.Release.IsUpgrade`: True if the operation is an upgrade
- `.Release.IsInstall`: True if the operation is an install
- `.Release.Revision`: Revision number
- `.Release.Service`: Service rendering the template (always "Helm")

#### The `.Chart` Object

Retrieves metadata from `Chart.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
  labels:
    helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
data:
  config.properties: |-
    chapterNumber={{ .Values.chapterNumber }}
    chapterName={{ .Values.chapterName }}
```

**Example Output**:
```yaml
$ helm template chart-example 05-Templating/examples/chart-example
---
# Source: chart-example/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: chart-example
  labels:
    helm.sh/chart: chart-example-1.0.0
    app.kubernetes.io/version: 0.1.0
data:
  config.properties: |-
    chapterNumber=6
    chapterName=Understanding Helm Templates
```

#### The `.Template` Object

Provides metadata about the current template:
- `.Template.Name`: File path of the template (e.g., `mychart/templates/mytemplate.yaml`)
- `.Template.BasePath`: Path leading to the templates directory (e.g., `mychart/templates`)

#### The `.Capabilities` Object

Provides cluster information:
- `.Capabilities.APIVersions.Has "batch/v1"`: Checks if API version is available
- `.Capabilities.KubeVersion`: Retrieves Kubernetes version

**Example**:
```yaml
{{ .Capabilities.APIVersions.Has "batch/v1" }}
{{ .Capabilities.KubeVersion }}
```

#### The `.Files` Object

Accesses files in the chart directory:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
data:
  config.properties: |-
    {{ .Files.Get "files/config.properties" }}
```

**Example Output**:
```yaml
$ helm template basic-files-example 05-Templating/examples/files-example/get
---
# Source: files-example/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: basic-files-example
data:
  config.properties: |-
    chapterNumber=6
```

**Functions**:
- `.Files.Get`: Retrieves file contents
- `.Files.Glob`: Matches files with a pattern (e.g., `files/*`)
- `.Files.AsConfig`: Returns files as YAML dictionaries
- `.Files.AsSecrets`: Returns files as Base64-encoded secrets

**Example with `.Files.Glob`**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
data:
  {{ (.Files.Glob "files/chapter*").AsConfig | indent 2 }}
---
kind: Secret
data:
  {{ (.Files.Glob "files/secret*").AsSecrets | indent 2 }}
```

**Example Output**:
```yaml
$ helm template glob-example 05-Templating/examples/files-example/glob
---
# Source: files-example/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: glob-example
data:
  chapter.properties: |
    chapterNumber=6
    chapterName=Understanding Helm Templates
---
# Source: files-example/templates/secret.yaml
apiVersion: v1
kind: Secret
data:
  secret.properties:
    dXNlcm5hbWU9bXl1c2VyMTIzCnBhc3N3b3JkPW15cGFzczEyMwo=
```

This summary provides a concise overview of built-in Helm objects with commands and examples included.
