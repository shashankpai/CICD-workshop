# Creating New Guestbook Helm Chart 

we will begin to explore Helm chart development by discussing the following topics:
- Understanding the Guestbook application
- Understanding the YAML format
- Using helm create to scaffold a new Helm chart
- Deploying the scaffolded Guestbook chart
- Exploring the Chart.yaml file
- Updating the Chart.yaml file


### Understanding the Guestbook Application

The Guestbook application is a simple PHP-based frontend that stores messages in a Redis backend. Here’s a quick overview of how it works:

- **Frontend**: Users enter a message in a dialog box and click Submit. The message is saved to Redis and displayed on the page.

- **Backend**: Redis, an in-memory key-value store, is used. It consists of a leader node for writing data and follower nodes for reading and displaying messages. 

The interaction between the Guestbook frontend and Redis is illustrated below:

![Guestbook Frontend and Redis Interaction](link-to-diagram)

In the next part, we'll develop a Helm chart for deploying the Guestbook application. To get started, we'll cover the basics of YAML format, which is essential for Helm chart development.

### Understanding the YAML Format

YAML (YAML Ain't Markup Language) is a human-readable file format used for configuration. It’s commonly used in Kubernetes and Helm charts. Here’s a brief overview:

#### Key-Value Pairs

In YAML, configuration is defined using key-value pairs. For example:

```yaml
name: LearnHelm
```

- **Key**: `name`
- **Value**: `LearnHelm`

**Important**: Ensure a space exists between the colon and the value. For example, `name:LearnHelm` is incorrect and will cause a parsing error.

#### Nested Elements

YAML supports complex configurations with nested elements:

```yaml
resources:
  limits:
    cpu: 100m
    memory: 512Mi
```

- **Key**: `resources.limits.cpu`
- **Value**: `100m`

Indentation (typically two spaces) indicates nested levels. **Note**: Tabs are not supported in YAML.

#### Value Types

Values can be of different types:

- **String**: Can be declared with or without quotation marks. Multiline strings use the pipe (|) symbol:

  ```yaml
  configuration: |
    server.port=8443
    logging.file.path=/var/log
  ```

- **Integer**: Numeric values without quotation marks:

  ```yaml
  replicas: 1
  ```

- **Boolean**: Declared with `true` or `false`, and other variants:

  ```yaml
  ingress:
    enable: true
  ```

- **List**: Items are listed with dashes (-):

  ```yaml
  servicePorts:
    - 8080
    - 8443
  ```

  Lists can also contain objects:

  ```yaml
  deployment:
    env:
      - name: MY_VAR
        value: MY_VALUE
      - name: SERVICE_NAME
        value: MY_SERVICE
  ```

#### JSON Format

YAML is a superset of JSON. JSON uses braces and brackets for configuration:

```json
{
  "deployment": {
    "env": [
      {
        "name": "MY_VAR",
        "value": "MY_VALUE"
      },
      {
        "name": "SERVICE_NAME",
        "value": "MY_SERVICE"
      }
    ]
  }
}
```

- **Curly Braces ({})**: Denote blocks.
- **Square Brackets ([])**: Denote lists.

With this foundation, you’re ready to work with Helm charts, which use YAML for configuration. In the next section, we'll start scaffolding our Guestbook Helm chart.

### Scaffolding the Guestbook Helm Chart

To start developing a new Helm chart, you can use the `helm create` command. This command generates a new project folder with basic Helm chart scaffolding. Here’s how to scaffold the Guestbook Helm chart:

#### Creating the Chart

1. **Run the `helm create` command**:
   ```bash
   $ helm create guestbook
   ```

2. **Review the generated files and directories**:
   ```bash
   $ ls -al guestbook
   ```
   You should see:
   - `charts/` – Directory for chart dependencies.
   - `Chart.yaml` – Metadata about the Helm chart.
   - `.helmignore` – Lists files and directories to omit from packaging.
   - `templates/` – Contains Golang templates for generating Kubernetes resources.
   - `values.yaml` – Default values for the chart.

   Inside the `templates/` directory:
   ```bash
   $ ls -l guestbook/templates/
   ```
   You should see:
   - `deployment.yaml`
   - `_helpers.tpl`
   - `hpa.yaml`
   - `ingress.yaml`
   - `NOTES.txt`
   - `serviceaccount.yaml`
   - `service.yaml`
   - `tests/` – Directory for test templates.

#### Understanding Generated Files

Here’s a high-level overview of the files created:

| File/Directory             | Definition                                                                 | Required? |
|----------------------------|-----------------------------------------------------------------------------|-----------|
| `charts/`                  | Contains dependencies or other Helm charts.                                | No        |
| `Chart.yaml`               | Contains metadata about the Helm chart.                                    | Yes       |
| `.helmignore`              | Lists files and directories to exclude from packaging.                      | No        |
| `templates/`               | Contains Golang templates for Kubernetes resources.                         | Yes       |
| `templates/*.yaml`         | Templates for generating Kubernetes resources.                              | Yes       |
| `templates/_*.tpl`         | Boilerplate helper templates.                                               | No        |
| `templates/NOTES.txt`      | Provides usage instructions after chart installation.                       | No        |
| `templates/tests/`         | Directory for grouping test templates.                                      | No        |
| `values.yaml`              | Default values for the chart.                                                | No, but recommended |

Additional files that can be included:

| File/Directory             | Definition                                                                  | Required? |
|----------------------------|------------------------------------------------------------------------------|-----------|
| `Chart.lock`               | Locks in previously applied dependency versions.                            | No        |
| `crds/`                    | Contains Custom Resource Definitions (CRDs) installed before other resources. | No        |
| `README.md`                | Contains installation and usage information.                                | No, but recommended |
| `LICENSE`                  | Contains the chart’s license information.                                    | No        |
| `values.schema.json`       | Contains the chart’s values schema for input validation.                      | No        |

#### Deploying the Scaffolded Chart

The `helm create` command generates a functional Helm chart. You can deploy it to see what gets installed. This provides a basic starting point to customize and extend the chart according to your needs.

### Setting Up and Installing the Helm Chart

#### 1. Start Minikube

In your terminal, start Minikube:

```sh
$ minikube start
```

#### 2. Create a Namespace

Create a new namespace named `guestbook`:

```sh
$ kubectl create namespace guestbook
```

#### 3. Install the Chart

Install the Helm chart by running the following command, where `./guestbook` represents the folder generated by `helm create`:

```sh
$ helm install guestbook ./guestbook -n guestbook
```

#### 4. Review the Deployed Resources

To review the resources deployed by the chart, use:

```sh
$ helm get manifest guestbook -n guestbook
```

This command will show you the resources that were deployed. For example, you might see:

- A service account
- A service
- A deployment

The default image deployed will be:

```yaml
image: "nginx:1.16.0"
```

#### 5. Access the NGINX Application

Since the Helm chart created a `ClusterIP` service, use `kubectl port-forward` to access the NGINX pod:

```sh
$ kubectl -n guestbook port-forward svc/guestbook 8080:80
```

In a new terminal window (since the current one will block while `kubectl port-forward` is active), use `curl` to reach NGINX:

```sh
$ curl localhost:8080
```

You should see:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
  body {
    width: 35em;
    margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif;
  }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
```

#### 6. Clean Up

1. **Stop Port-Forwarding**

   Press `Ctrl + c` in the terminal window running the `kubectl port-forward` command or close the terminal window.

2. **Uninstall the Helm Release**

   Uninstall the `guestbook` Helm release:

   ```sh
   $ helm uninstall guestbook -n guestbook
   ```

3. **Delete the Namespace**

   Delete the `guestbook` namespace:

   ```sh
   $ kubectl delete namespace guestbook
   ```

4. **Shut Down Minikube**

   Stop your Minikube cluster:

   ```sh
   $ minikube stop
   ```

---

### Understanding the Chart.yaml File

The `Chart.yaml` file, also known as the chart definition, contains metadata about a Helm chart. Here's a detailed look at its fields:

#### Required Fields

- **apiVersion**: The chart API version (e.g., `v2`).
- **name**: The name of the Helm chart.
- **version**: The version of the Helm chart in SemVer format.

#### Optional Fields

- **description**: A brief description of the Helm chart.
- **type**: The type of Helm chart (`Application` or `Library`).
- **appVersion**: The version of the application that the Helm chart deploys.

#### Additional Fields

- **kubeVersion**: A range of compatible Kubernetes versions.
- **keywords**: A list of keywords for searching the chart.
- **home**: The URL to the Helm chart’s home page.
- **sources**: A list of URLs linking to the source code.
- **dependencies**: A list of charts that the Helm chart depends on.
- **maintainers**: A list of Helm chart maintainers.
- **icon**: An icon representing the Helm chart.
- **deprecated**: Indicates whether the chart is deprecated.
- **annotations**: Custom metadata for the Helm chart.

#### Updating the Guestbook Chart.yaml

Update the `Chart.yaml` file for the `guestbook` chart with the following changes:

```yaml
apiVersion: v2
name: guestbook
description: An application used for keeping a running record of guests
version: 1.0.0
appVersion: v5
```

With these updates, your `Chart.yaml` file should accurately reflect the application that you are deploying.

