# Dependency Management 

we will cover the following topics:
• Declaring chart dependencies
• The dependencies map
• Downloading chart dependencies
• Conditional dependencies
• Altering dependency names and values
• Updating the guestbook Helm chart
• Cleaning up

The Helm chart we are developing, guestbook, will deploy two primary components. The first is a Redis backend, which is used to persist a list of messages. The second component is the frontend, where the user enters messages in a text box.

To deploy Redis, you might think that extensive modifications are required to the existing guestbook chart. However, this isn't necessarily the case. There are many Redis Helm charts available in Artifact Hub. We can use Helm's dependency management features to declare one of those charts as a dependency. When the guestbook chart is installed in a Kubernetes cluster, the Redis dependency will also be installed. By declaring Redis as a dependency, we can significantly reduce the effort needed to create the backend for our application.

We'll now explore how Helm manages dependencies and then use this knowledge to incorporate a Redis dependency into our Helm chart.

### Declaring Helm Chart Dependencies

Chart dependencies allow one Helm chart (the parent chart) to install resources from another chart. For instance, when you installed WordPress in a previous example, the MariaDB database was included as a dependency, not as a native resource. 

You can view the dependencies for any chart using the `helm show chart` command. For example:

```bash
$ helm show chart bitnami/wordpress --version 12.1.4
```

You'll see a `dependencies` section in the output, listing MariaDB, Memcached, and a library chart called `common`. 

### Dependency Map in Chart.yaml

The `dependencies` map in `Chart.yaml` defines the dependencies for a chart. The essential fields include:

- **name**: The name of the dependency chart.
- **repository**: The location of the dependency chart.
- **version**: The version of the dependency chart.

Optional fields include:

- **condition**: A Boolean value to include or exclude the dependency.
- **tags**: List of tags to control inclusion.
- **import-values**: Maps values from the dependency to the parent chart.
- **alias**: Allows renaming the dependency.

### Downloading Chart Dependencies

You can manage dependencies using Helm’s `helm dependency` commands:

- **list**: Lists dependencies for the chart.
- **update**: Downloads dependencies listed in `Chart.yaml` and generates a `Chart.lock` file.
- **build**: Downloads dependencies listed in `Chart.lock`. If `Chart.lock` is missing, it behaves like `update`.

#### Example Usage:

1. **List Dependencies**:
    ```bash
    $ helm dependency list 04-Dependency-Management/examples/basic-fields
    ```
    This shows the dependencies declared in the chart, including their download status.

2. **Download Dependencies**:
    ```bash
    $ helm dependency update 04-Dependency-Management/examples/basic-fields
    ```
    After running this command, the status will update to `ok`, and the dependencies will be saved in a `charts/` directory.

3. **View Downloaded Dependencies**:
    ```bash
    $ ls 04-Dependency-Management/examples/basic-fields/charts
    ```
    This will list the downloaded `.tgz` files, which are the dependency charts.

4. **Understanding Chart.lock**:
    ```bash
    $ cat 04-Dependency-Management/examples/basic-fields/Chart.lock
    ```
    The `Chart.lock` file locks in the dependency versions, ensuring consistency across different environments.

5. **Wildcard Versions**:
    If you specify a version as `9.x.x`, Helm downloads the latest minor and patch versions. After downloading, the exact version is locked in `Chart.lock`:

    ```bash
    $ helm dependency update 04-Dependency-Management/examples/wildcard-version
    ```

    The resulting `Chart.lock` file will lock in the specific version, such as `9.8.1`.

### Rebuilding Dependencies

If you delete the `charts/` directory and run `helm dependency build`, it will redownload the versions specified in `Chart.lock`:

```bash
$ rm -rf 04-Dependency-Management/examples/wildcard-version/charts
$ helm dependency build 04-Dependency-Management/examples/wildcard-version
```

This ensures that the exact versions locked in `Chart.lock` are restored.

### Creating Conditional Dependencies in Helm

Helm allows conditional dependencies using the `condition` and `tags` fields in the `dependencies` section of `Chart.yaml`. Let's explore how these work using examples from `04-Dependency-Management/examples/condition-example` and `04-Dependency-Management/examples/tags-example`.

#### 1. Using the `condition` Field

The `condition` field is used to toggle dependencies based on Boolean values. Here's an example:

- **Chart.yaml** (`04-Dependency-Management/examples/condition-example/Chart.yaml`):
    ```yaml
    dependencies:
    - name: mariadb
      repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
      version: 9.5.0
      condition: mariadb.enabled
    ```
  The `condition: mariadb.enabled` setting allows toggling the MariaDB dependency. If `mariadb.enabled` is set to `true`, MariaDB is included; otherwise, it is excluded.

- **values.yaml** (`04-Dependency-Management/examples/condition-example/values.yaml`):
    ```yaml
    mariadb:
      enabled: true
    ```
  Since `mariadb.enabled` is `true` by default, MariaDB will be included.

#### Steps to Verify:

1. **Update dependencies**: 
    ```bash
    helm dependency update 04-Dependency-Management/examples/condition-example
    ```

2. **Install the chart**:
    ```bash
    helm install conditional-example 04-Dependency-Management/examples/condition-example -n 04-Dependency-Management
    ```

3. **Check if MariaDB resources were created**:
    ```bash
    helm get manifest conditional-example -n 04-Dependency-Management | grep mariadb
    ```

4. **Disable MariaDB and upgrade**:
    ```bash
    helm upgrade conditional-example 04-Dependency-Management/examples/condition-example --set mariadb.enabled=false -n 04-Dependency-Management
    ```

5. **Verify MariaDB exclusion**:
    ```bash
    helm get manifest conditional-example -n 04-Dependency-Management | grep mariadb
    ```

#### 2. Using the `tags` Field

The `tags` field is used to enable/disable multiple dependencies using descriptive labels.

- **Chart.yaml** (`04-Dependency-Management/examples/tags-example/Chart.yaml`):
    ```yaml
    dependencies:
    - name: mariadb
      repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
      version: 9.5.0
      tags:
        - backend
        - database
    - name: memcached
      repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
      version: 5.15.6
      tags:
        - backend
        - cache
    ```

- **values.yaml** (`04-Dependency-Management/examples/tags-example/values.yaml`):
    ```yaml
    tags:
      backend: true
    ```
  Since the `backend` tag is enabled, both `mariadb` and `memcached` will be included.

#### Steps to Verify:

1. **Upgrade with the `tags-example` chart**:
    ```bash
    helm upgrade conditional-example 04-Dependency-Management/examples/tags-example -n 04-Dependency-Management
    ```

2. **Check if both `mariadb` and `memcached` were installed**:
    ```bash
    helm get manifest conditional-example -n 04-Dependency-Management | grep mariadb
    helm get manifest conditional-example -n 04-Dependency-Management | grep memcached
    ```

3. **Exclude `memcached` by overriding tags**:
    ```bash
    helm upgrade conditional-example 04-Dependency-Management/examples/tags-example --set tags.backend=false --set tags.database=true -n 04-Dependency-Management
    ```

4. **Verify `mariadb` inclusion and `memcached` exclusion**:
    ```bash
    helm get manifest conditional-example -n 04-Dependency-Management | grep mariadb
    helm get manifest conditional-example -n 04-Dependency-Management | grep memcached
    ```

### Conclusion

- **Conditions**: Use `chartname.enabled` for individual dependencies.
- **Tags**: Use to enable/disable multiple dependencies with descriptive labels.
- **Note**: If `condition` and `tags` are both used, `condition` overrides `tags`.

Finally, uninstall the release:

```bash
helm uninstall conditional-example -n 04-Dependency-Management
```

### Modifying Dependency Names and Values

When incorporating a dependency within a Helm chart, altering its values is often necessary. The most straightforward way to do this is by overriding values under a map that matches the dependency's name.

For instance, consider the **basic-fields** chart located in `04-Dependency-Management/examples/basic-fields`. This chart includes a MariaDB dependency in its `Chart.yaml`:

```yaml
dependencies:
  - name: mariadb
    repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
    version: 9.5.0
```

To override values like `image.registry`, `image.repository`, and `image.tag`, you can structure your `values.yaml` like this:

```yaml
mariadb:
  image:
    registry: my-registry.example.com
    repository: my-mariadb
    tag: my-tag
```

#### Hands-On Example:
1. Install the `basic-fields` chart and override MariaDB’s image tag:
   ```bash
   helm install override-example 04-Dependency-Management/examples/basic-fields --set mariadb.image.tag=latest -n 04-Dependency-Management
   ```
2. Verify the change:
   ```bash
   helm get manifest override-example -n 04-Dependency-Management | grep latest
   ```
3. Uninstall the release:
   ```bash
   helm uninstall override-example -n 04-Dependency-Management
   ```

#### Using Alias

The `alias` setting allows you to use the same dependency multiple times with different configurations. For example, the **alias-example** chart, located in `04-Dependency-Management/examples/alias-example`, includes two MariaDB dependencies with aliases:

```yaml
dependencies:
  - name: mariadb
    repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
    version: 9.5.0
    alias: db1
  - name: mariadb
    repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
    version: 9.5.0
    alias: db2
```

Each MariaDB instance can have different configurations using their aliases.

1. Install the chart and override the image tags:
   ```bash
   helm install alias-example 04-Dependency-Management/examples/alias-example --set db1.image.tag=latest --set db2.image.tag=10.4 -n 04-Dependency-Management
   ```
2. Verify the tags:
   ```bash
   helm get manifest alias-example -n 04-Dependency-Management | grep latest
   helm get manifest alias-example -n 04-Dependency-Management | grep 10.4
   ```
3. Uninstall the release:
   ```bash
   helm uninstall alias-example -n 04-Dependency-Management
   ```

#### Importing Values

The `import-values` setting in the `Chart.yaml` allows altering how dependency values are propagated to the parent chart. It can be used in two formats: `exports` and `child-parent`.

**Exports Format Example:**
If a dependency has the following `values.yaml`:

```yaml
exports:
  image:
    registry: my-registry
    repository: my-repository
    tag: my-tag
```

You can import these values in the parent chart:

```yaml
dependencies:
  - name: dependency
    repository: http://localhost:8080
    version: 1.0.0
    import-values:
      - image
```

This results in the following propagation:

```yaml
registry: my-registry
repository: my-repository
tag: my-tag
```

**Child-Parent Format Example:**
If a dependency has deeply nested values:

```yaml
common:
  deployment:
    image:
      registry: my-registry
      repository: my-repository
      tag: my-tag
```

You can import these in the parent chart:

```yaml
dependencies:
  - name: dependency
    repository: http://localhost:8080
    version: 1.0.0
    import-values:
      - child: common.deployment.image
        parent: image
```

Resulting in:

```yaml
image:
  registry: my-registry
  repository: my-repository
  tag: my-tag
```

Note: When using `import-values`, you cannot override the imported values. Overrides should be done separately.

With this understanding, you can now update your **guestbook** chart with a Redis dependency to create the backend.

### Updating the Guestbook Helm Chart with Redis Dependency

To add Redis as a dependency for your **guestbook** Helm chart, follow these steps:

1. **Search for Redis Chart:**
   - Use Artifact Hub to search for Redis charts:
     ```bash
     helm search hub redis
     ```
   - The first result will likely be Bitnami's Redis chart, which we'll use as our dependency. If you haven't added the Bitnami repository yet, add it now:
     ```bash
     helm repo add bitnami https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
     ```

2. **Determine the Redis Chart Version:**
   - List available Redis chart versions:
     ```bash
     helm search repo redis --versions
     ```
   - We'll use the wildcard version `15.5.x` to ensure we get the latest patch release while allowing future updates.

3. **Update the guestbook Chart:**
   - Modify your `guestbook` chart’s `Chart.yaml` to declare the Redis dependency with a conditional property to enable or disable Redis:
     ```yaml
     dependencies:
       - name: redis
         repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
         version: 15.5.x
         condition: redis.enabled
     ```
   - Your updated `Chart.yaml` should look like this:
     ```yaml
     apiVersion: v2
     name: guestbook
     description: An application used for keeping a running record of guests
     type: application
     version: 0.1.0
     appVersion: v5
     dependencies:
       - name: redis
         repository: https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
         version: 15.5.x
         condition: redis.enabled
     ```

4. **Install Redis Dependency:**
   - Download the latest Redis 15.5.x release:
     ```bash
     helm dependency update guestbook
     ```
   - Deploy the `guestbook` chart in your Minikube environment:
     ```bash
     helm install guestbook guestbook -n 04-Dependency-Management
     ```

5. **Verify Installation:**
   - Check that the Redis StatefulSets were created:
     ```bash
     kubectl get statefulsets -n 04-Dependency-Management
     ```
   - You should see outputs similar to:
     ```bash
     NAME                      READY   AGE
     guestbook-redis-master    1/1     3m24s
     guestbook-redis-replicas  3/3     3m24s
     ```

6. **Cleanup:**
   - Delete the `04-Dependency-Management` namespace:
     ```bash
     kubectl delete namespace 04-Dependency-Management
     ```
   - Stop your Minikube environment:
     ```bash
     minikube stop
     ```

### Summary

Using Helm’s dependency management significantly reduces the effort required to deploy complex applications. In our guestbook example, adding Redis as a backend only took a few lines in the `Chart.yaml` file, simplifying the process considerably compared to writing a Redis chart from scratch.

