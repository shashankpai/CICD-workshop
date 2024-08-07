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


