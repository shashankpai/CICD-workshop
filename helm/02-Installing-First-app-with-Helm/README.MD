# Installing first wordpress app with HELM

we will cover the following main topics:
• Understanding the WordPress application
• Finding a WordPress chart
• Creating a Kubernetes environment
• Installing a WordPress chart
• Choosing between --set and --values
• Accessing the WordPress application
• Upgrading the WordPress release
• Rolling back the WordPress release
• Uninstalling the WordPress release
• Shutting down your environment


## Understanding WordPress application

WordPress is an open-source Content Management System (CMS) widely used for creating websites and blogs. There are two main versions: WordPress.com and WordPress.org. WordPress.com is a Software-as-a-Service (SaaS) version where the WordPress application and its components are hosted and managed by WordPress. Users can access pre-configured instances without needing to install anything.

In contrast, WordPress.org is a self-hosted option that requires users to set up and maintain their WordPress instances, which demands a certain level of expertise.

While WordPress.com might seem more convenient due to its ease of use, it has several limitations compared to the self-hosted WordPress.org, such as:
- Fewer features available
- Limited control over the website
- Additional costs for premium features
- Inability to modify the backend code

On the other hand, WordPress.org offers complete control over the website and access to the full range of WordPress features, from plugin installation to backend code customization.

Setting up a self-hosted WordPress instance involves deploying several components. The platform requires a MySQL or MariaDB database to store website and administrative data, and the WordPress interface runs as a PHP frontend. In a Kubernetes environment, deploying these components entails creating various resources, including:
- Secrets for database and admin console authentication
- A ConfigMap for externalized database configuration
- **Services** for networking
- **PersistentVolumeClaim** for database storage
- **StatefulSet** for deploying the database in a stateful manner
- **Deployment** for deploying the frontend

Setting up these resources demands expertise in both WordPress and Kubernetes. WordPress knowledge is necessary to understand and 
configure the essential components, while Kubernetes expertise is needed to deploy these components as Kubernetes resources. 
Given the complexity and the number of elements involved, deploying WordPress on Kubernetes can be challenging.

This is where Helm comes in as an ideal solution. Instead of manually creating and configuring each Kubernetes resource, 
Helm can be used as a package manager to streamline the deployment and configuration of WordPress on Kubernetes. 
We'll start by using Artifact Hub to find a suitable WordPress Helm chart, then proceed to deploy WordPress to your Kubernetes cluster using Helm, 
exploring its core features along the way.

## Finding a WordPress Chart

Helm charts can be shared by publishing them to a chart repository, which is essentially a storage location for packaged charts. 
Repositories can be hosted on HTTP servers, such as GitHub Pages, Amazon S3, or simple web servers like Apache HTTPD. 
Recently, OCI registries like Docker Hub and Quay also support Helm charts.

To use charts from a repository, you need to configure Helm to access it using the helm repo add command. However, with numerous repositories available, 
finding the right one can be challenging. The Kubernetes community created Artifact Hub to simplify this process.

Artifact Hub is a centralized platform for Kubernetes artifacts, including Helm charts, operators, and plugins. In this chapter, 
we’ll use Artifact Hub to locate a WordPress Helm chart, add its repository, and install it. You can interact with Artifact Hub via the command line or a 
web browser. Command line searches return URLs to Artifact Hub for more details and repository addition instructions.

## Searching for WordPress charts from the command line

Searching Artifact Hub:
To search for charts on Artifact Hub, use the command:

```
helm search hub
```
To search for a keyword within added repositories, use the command:

```
helm search repo
```

If no repositories have been added yet, first use the helm search hub command to find available Helm charts across all public repositories. 
After adding a repository, you can then use helm search repo to search within those repositories.

Now, let's search Artifact Hub for WordPress charts. Each chart on Artifact Hub has associated keywords that you can search for. 
To find charts related to WordPress, run:

```
helm search hub wordpress
```
As you may have noticed, some of the values returned are truncated. This is because the default
output of helm search hub is table, causing the results to be returned in tabular format. By
default, columns wider than 50 characters are truncated. This truncation can be avoided by using the
--max-col-width=0 flag.
Try running the following command by including the --max-col-width flag to view the untruncated
results in tabular format:

```
helm search hub wordpress --max-col-width=0
```

Try running the previous command again with the --output yaml flag:
```
helm search hub wordpress --output yaml
```

### Adding full bitnami repo

Now that we understand how to work with charts in the Bitnami repository, let’s add the repository, ensuring we can specify specific chart versions without worrying about them being removed in the future. The only change to the `helm repo add` command is the repository URL.

Here’s how to add the repository and verify access:

1. **Add the full Bitnami chart repository:**  
   ```bash
   $ helm repo add bitnami https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
   ```

2. **Verify the repository has been added by listing the repositories:**  
   ```bash
   $ helm repo list
   ```
   The output should confirm the Bitnami repository has been added:
   ```bash
   NAME     URL
   bitnami  https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami
   ```

3. **Search for charts with the `bitnami` keyword in your locally configured repositories:**  
   ```bash
   $ helm search repo bitnami --output yaml
   ```

Like the `helm search hub` command, the `helm search repo` command also takes a keyword as an argument. For example, using `bitnami`
 as a keyword will return all charts in the Bitnami repository, as well as any other charts that include the `bitnami` keyword.

To verify that you have access to the WordPress chart, run the following command with `wordpress` as the argument:
```bash
$ helm search repo wordpress
```
This will display the WordPress chart you found on Artifact Hub and viewed in your browser.

The value in the `NAME` field before the slash (`/`) indicates the name of the repository containing the returned Helm chart. For example, the latest version of the WordPress chart from the Bitnami repository at the time of writing is version 12.1.6, which will be used for the installation.

To view previous versions of the chart, you can use the `--versions` flag with the search command:
```bash
$ helm search repo wordpress --versions
```
You can find detailed information about a Helm chart on its Artifact Hub page. Once the chart’s repository is added locally, you can access this information and more directly from the command line using the following `helm show` subcommands:

- **Display the chart’s metadata:**
  ```bash
  helm show chart
  ```

- **Display the chart’s README file:**
  ```bash
  helm show readme
  ```

- **Display the chart’s values:**
  ```bash
  helm show values
  ```

- **Display the chart’s metadata, README, and values all at once:**
  ```bash
  helm show all
  ```

Let's apply these commands to the Bitnami WordPress chart. In each command, refer to the chart as `bitnami/wordpress`. To retrieve information about a specific version, use the `--version` flag. For example, to get details on version 12.1.6, use the following command:

```bash
$ helm show chart bitnami/wordpress --version 12.1.6
```

If the `--version` flag is omitted, the latest version of the chart will be used.


## Bring up the kubernetes environment

To create a Kubernetes environment , we will use minikube.

1. Start your Kubernetes cluster by running the following command:
   ```bash
   $ minikube start
   ```
2. After a short while, you should see a line in the output similar to:
   ```
   Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
   ```
3. Once the Minikube cluster is up and running, create a dedicated namespace for this exercise. Run the following command to create a namespace called `wordpress`:
   ```bash
   $ kubectl create namespace wordpress
   ```
Now that the cluster setup is complete, let's proceed with installing the WordPress chart on your Kubernetes cluster.

## Installing a WordPress chart


### Creating a Values File for Configuration

You can customize the configuration of a Helm chart by providing a YAML-formatted values file. To do this, you'll first need to inspect the chart’s available values. This can be done using the `helm show values` command.

Run the following command to view the configurable values for the WordPress chart:
```bash
$ helm show values bitnami/wordpress --version 12.1.6
```

This command will produce a lengthy list of possible values that you can customize, many of which already have default settings.

As you can see, these values are used to set up a WordPress user and name your blog. To customize them, create a values file. Start by creating a new file on your machine called `wordpress-values.yaml`. Add the following content to the file:

```yaml
wordpressUsername: helm-user
wordpressPassword: my-password
wordpressEmail: helm-user@example.com
wordpressFirstName: Helm_is
wordpressLastName: Fun
wordpressBlogName: Learn Helm!
```

Feel free to modify these values as you like. Before proceeding with the installation, there's one more important value to add to your `wordpress-values.yaml` file, as shown in the next steps.

After adding this final value, your complete `wordpress-values.yaml` file should look like this:

```yaml
wordpressUsername: helm-user
wordpressPassword: my-password
wordpressEmail: helm-user@example.com
wordpressFirstName: Helm_is
wordpressLastName: Fun
wordpressBlogName: Learn Helm!
service:
  type: NodePort
```

Now that your values file is ready, let's proceed with the installation.

### Running the Installation

To install a Helm chart, use the `helm install` command. The basic syntax is:

```bash
helm install [NAME] [CHART] [flags]
```

- **NAME**: This is the name you assign to your Helm release. A release encapsulates the Kubernetes resources installed with a chart and tracks the application's life cycle. We will delve into how releases work later in this chapter.

- **CHART**: This is the name of the Helm chart you want to install. Charts from a repository can be installed using the format `<repo name>/<chart name>`.

- **flags**: The flags option allows you to further customize the installation, such as defining and overriding values or specifying the namespace. You can view all available flags by running `helm install --help`. This command, like others, supports the `--help` flag to display usage information and supported options.

Now that we understand how to use the `helm install` command, let's run the following command:

```bash
$ helm install wordpress bitnami/wordpress --values=wordpress-values.yaml --namespace wordpress --version 12.1.6
```

This command installs a new release called `wordpress` using the `bitnami/wordpress` Helm chart. It customizes the installation based on the values defined in the `wordpress-values.yaml` file and installs the chart in the `wordpress` namespace. The chart is deployed with version 12.1.6 as specified by the `--version` flag. If the `--version` flag is omitted, Helm will install the latest cached version of the chart.

### Inspecting Your Release

To inspect a Helm release and verify its installation, use the `helm list` command to list all releases in a given namespace.

Run the following command to view the list of releases in the `wordpress` namespace:
```bash
$ helm list --namespace wordpress
```
You should see only one release in this namespace, providing the following details:
- Release name
- Namespace
- Latest revision number
- Timestamp of the latest revision
- Release status

To get more detailed information about the release, you can use the following commands:

- **List hooks:**  
  ```bash
  $ helm get hooks wordpress --namespace wordpress
  ```
  Hooks are actions Helm performs during certain phases of the application’s life cycle. This installation did not create any hooks.

- **Get manifest:**  
  ```bash
  $ helm get manifest wordpress --namespace wordpress
  ```
  This command returns the Kubernetes resources created during installation, such as Secrets, ConfigMaps, PersistentVolumeClaim, Services, Deployment, StatefulSet, and ServiceAccount.

- **Get notes:**  
  ```bash
  $ helm get notes wordpress --namespace wordpress
  ```

- **Get values:**  
  ```bash
  $ helm get values wordpress --namespace wordpress
  ```

- **Get all information:**  
  ```bash
  $ helm get all wordpress --namespace wordpress
  ```

The `helm get manifest` command will show the Kubernetes manifests for:
- Two Secrets (for MariaDB and WordPress credentials)
- Two ConfigMaps (one for WordPress configuration and one for testing)
- One PersistentVolumeClaim (for MariaDB data)
- Two Services (for MariaDB and WordPress)
- One Deployment (for WordPress)
- One StatefulSet (for MariaDB)
- One ServiceAccount (for MariaDB)

This output helps you see where your values impacted the Kubernetes resources, such as setting the WordPress service type to NodePort.

### Choosing Between `--set` and `--values`

When installing WordPress, we used the `--values` flag to pass parameters to the Helm chart. There are two main ways to provide values:

- **`--set`**: Use this flag to pass values directly from the command line. It's suitable for simple values or a few configurations. However, it can be cumbersome for complex values like lists or maps.

- **`--values`**: Use this flag to specify values from a YAML file or URL. This method is preferred because it handles multiple values more easily and allows you to manage configurations in a file, which can be stored in a Source Code Management (SCM) system like Git for reproducibility.

**Note**: Sensitive values, such as passwords, should not be stored in source control. For such cases, use the `--set` flag to avoid committing secrets.

Other related flags include:

- **`--set-file`**: Passes a file with values in `key1=val1` format.
- **`--set-string`**: Treats all values provided in `key1=val1` format as strings.

Next, let’s explore the WordPress application we just installed.

### Accessing the WordPress Application

The release notes for the WordPress chart provide commands to access your application. Run the following commands:

For macOS or Linux:
```bash
export NODE_PORT=$(kubectl get --namespace wordpress -o jsonpath="{.spec.ports[0].nodePort}" services wordpress)
export NODE_IP=$(kubectl get nodes --namespace wordpress -o jsonpath="{.items[0].status.addresses[0].address}")
echo "WordPress URL: http://$NODE_IP:$NODE_PORT/"
echo "WordPress Admin URL: http://$NODE_IP:$NODE_PORT/admin"
```

These commands set environment variables for the NodePort and Node IP, and then display the URLs to access WordPress and its admin interface.

After defining the two environment variables based on the series of kubectl queries, the resulting
echo commands will reveal the URLs to access WordPress. The first URL is to view the home page
and is where visitors will access your site. The second URL is to reach the admin console, which is
used by website administrators to configure and manage the site’s content.

### Upgrading the WordPress Release

Upgrading a release involves modifying its values or updating the chart. In this example, we'll enhance the WordPress installation by adding new values.

**Modifying the Helm Values**

To improve availability, you may want to run multiple replicas of your application. The `replicaCount` value in the chart configuration controls this. To increase the number of WordPress replicas from 1 to 2, add the following to your `wordpress-values.yaml` file:
```yaml
replicaCount: 2
```

Next, configure resource requests. The chart’s `resources` map lets you set memory and CPU requests. Modify these values to set memory to 256Mi and CPU to 100m:
```yaml
resources:
  requests:
    memory: 256Mi
    cpu: 100m
```

Your updated `wordpress-values.yaml` should look like this:
```yaml
wordpressUsername: helm-user
wordpressPassword: my-password
wordpressEmail: helm-user@example.com
wordpressFirstName: Helm_is
wordpressLastName: Fun
wordpressBlogName: Learn Helm!
service:
  type: NodePort
replicaCount: 2
resources:
  requests:
    memory: 256Mi
    cpu: 100m
```

After updating the `wordpress-values.yaml` file, you can proceed with the `helm upgrade` command to apply these changes.
### Running the Upgrade

The `helm upgrade` command is similar to `helm install` in syntax:
```bash
helm upgrade [RELEASE] [CHART] [flags]
```
Here, `RELEASE` is the existing release name to upgrade. The `--install` flag will install the chart if the release doesn’t exist.

To upgrade the WordPress release with new values, use:
```bash
helm upgrade wordpress bitnami/wordpress --values wordpress-values.yaml -n wordpress --version 12.1.6
```
After running this command, you should see output similar to `helm install`, with the `REVISION` field updated to 2.

To verify, check the WordPress pods:
```bash
kubectl get pods -n wordpress
```
New pods will appear if the pod template was modified during the upgrade. Use `helm get manifest` and `kubectl get deployment` to observe changes.

### Reusing and Resetting Values During an Upgrade

In addition to `--set` and `--values`, `helm upgrade` includes:
- `--reuse-values`: Reuses the last release's values if no new values are provided.
- `--reset-values`: Resets to default values if new values are provided.

**Example 1: Upgrade without specifying values:**
```bash
helm upgrade wordpress bitnami/wordpress -n wordpress --version 12.1.6
```
Check values used:
```bash
helm get values wordpress -n wordpress
```
Values should match the previous upgrade.

**Example 2: Upgrade with specific values:**
```bash
helm upgrade wordpress bitnami/wordpress --set replicaCount=1 --set wordpressUsername=helm-user --set wordpressPassword=my-password -n wordpress --version 12.1.6
```
Check values after upgrade:
```bash
helm get values wordpress -n wordpress
```
The output will show only the provided values, with others reset to defaults.

**Tip:** For consistency, manage all values in a `values.yaml` file. This approach is clearer and easier to manage during upgrades.

With the commands provided, you should have multiple revisions of the WordPress release. If needed, you can roll back to a stable version with your desired settings.

### Rolling Back the WordPress Release

Rolling back a release is useful when reverting to a previous version of your application. Here's how to roll back the WordPress release:

#### Inspecting Release History

Helm keeps a history of revisions for each release. Revisions track values, Kubernetes resources, and chart versions used in each release. These revisions are stored in Kubernetes Secrets by default.

To list secrets and view revisions:
```bash
kubectl get secrets -n wordpress
```

You should see secrets like:
- `sh.helm.release.v1.wordpress.v1`
- `sh.helm.release.v1.wordpress.v2`
- `sh.helm.release.v1.wordpress.v3`
- `sh.helm.release.v1.wordpress.v4`

To view the revision history:
```bash
helm history wordpress -n wordpress
```

This command displays each revision’s number, status, and description. 

To check values for a specific revision:
```bash
helm get values wordpress --revision 3 -n wordpress
```

#### Performing the Rollback

Use the `helm rollback` command to revert to a previous revision:
```bash
helm rollback wordpress 3 -n wordpress
```

After executing the rollback, you should see a new revision with a status of `deployed`. Confirm the rollback by checking the release values:
```bash
helm get values wordpress -n wordpress
```

#### Uninstalling the WordPress Release

To uninstall and delete the WordPress release:
```bash
helm uninstall wordpress -n wordpress
```

Verify the release is removed:
```bash
helm list -n wordpress
```

Check for remaining resources:
```bash
kubectl get deployments -l app.kubernetes.io/name=wordpress -n wordpress
```

If a PersistentVolumeClaim remains, delete it manually:
```bash
kubectl delete pvc data-wordpress-mariadb-0 -n wordpress
```
