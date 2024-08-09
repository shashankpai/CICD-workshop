
# Approaches to resource management

In order to deploy an application on Kubernetes, we need to interact with the Kubernetes API to create
resources. kubectl is the tool we use to talk to the Kubernetes API. kubectl is a command-line
interface (CLI) tool used to abstract the complexity of the Kubernetes API from end users, allowing
them to more efficiently work on the platform.

Let’s discuss how kubectl can be used to manage Kubernetes resources.

### Imperative and declarative configurations

The kubectl tool provides a series of subcommands to create and modify resources in an imperative
fashion. Here is a small list of these commands:
• create
• describe
• edit
• delete

The kubectl commands follow a common format, as shown here:

``` kubectl <verb> <noun> <arguments> ```

The verb refers to one of the kubectl subcommands, and the noun refers to a particular Kubernetes
resource. For example, the following command can be run to create a deployment:

``` kubectl create deployment my-deployment --image=busybox ```

This would instruct kubectl to talk to the Deployment API endpoint and create a new deployment
called my-deployment, using the busybox image from Docker Hub.
You could use kubectl to get more information on the deployment that was created by using the
describe subcommand, as follows:

```kubectl describe deployment my-deployment```

This command would retrieve information about the deployment and format the result in a readable
format that allows developers to inspect the live my-deployment deployment on Kubernetes.

If a change to the deployment was desired, a developer could use the edit subcommand to modify
it in place, like this:

```kubectl edit deployment my-deployment```

This command would open a text editor, allowing you to modify the deployment.

When it comes to deleting a resource, the user could run the delete subcommand, as illustrated here:

```kubectl delete deployment my-deployment```

This would call the appropriate API endpoint to delete the my-deployment deployment.

Kubernetes resources, once created, exist in the cluster as JavaScript Object Notation (JSON) resource
files, which can be exported as YAML Ain’t Markup Language (YAML) files for greater human readability.
An example resource in YAML format can be seen here:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
labels:
        app: busybox
    spec:
      containers:
        - name: main
          image: busybox
          args:
            - sleep
            - infinity
```

The preceding YAML format presents a very basic use case. It deploys the busybox image from
Docker Hub and runs the sleep command indefinitely to keep the Pod running.

While it may be easier to create resources imperatively using the kubectl subcommands we have
just described, Kubernetes allows you to directly manage the YAML resources in a declarative fashion
to gain more control over resource creation. The kubectl subcommands do not always let you
configure all the possible resource options, but creating YAML files directly allows you to more flexibly
create resources and fill in the gaps that the kubectl subcommands may contain.

When creating resources declaratively, users first write out the resource they want to create in YAML
format. Next, they use the kubectl tool to apply the resource against the Kubernetes API. While in
imperative configuration developers use kubectl subcommands to manage resources, declarative
configuration relies primarily on only one subcommand—apply.
Declarative configuration often takes the following form:
```
kubectl apply -f my-deployment.yaml
```
This command gives Kubernetes a YAML resource that contains a resource specification, although
the JSON format can be used as well. Kubernetes infers the action to perform on resources (create or
modify) based on whether or not they exist.

### An application may be configured declaratively by following these steps:

1. First, the user can create a file called deployment.yaml and provide a YAML-formatted
specification for the deployment. We will use the same example as before, as follows:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
spec:
  replicas: 1
selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
        - name: main
          image: busybox
          args:
            - sleep
            - infinity
```
2. A deployment can then be created with the following command:
```
kubectl apply –f deployment.yaml
```
Upon running this command, Kubernetes will attempt to create a deployment in the way
you specified.

3. If you wanted to make a change to the deployment by changing the number of replicas to 2,
you would first modify the deployment.yaml file, as follows:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
spec:
  replicas: 2
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
        - name: main
          image: busybox
          args:
            - sleep
            - infinity
```

4. You would then apply the change with kubectl apply, like this:

```
kubectl apply –f deployment.yaml
```
After running that command, Kubernetes would apply the provided deployment declaration
over the previously applied deployment. At this point, the application would scale up from a
replica value of 1 to 2.

5. When it comes to deleting an application, the Kubernetes documentation actually recommends
doing so in an imperative manner; that is, using the delete subcommand instead of apply,
as illustrated here:
```
kubectl delete –f deployment.yaml
```
As you can see, the delete subcommand uses the –f flag to delete the resource from the
given file.

With an understanding of how Kubernetes resources are created, let’s now discuss some of the challenges
involved in resource configuration.

### The many types of Kubernetes resources

First of all, as described in the Deploying a Kubernetes application section, there are many different
types of resources in Kubernetes. In order to be effective on Kubernetes, developers need to be able
to determine which resources are required to deploy their applications, and they need to understand
them at a deep enough level to configure them appropriately. This requires a lot of knowledge of and
training on the platform. While understanding and creating resources may already sound like a large
hurdle, this is actually just the beginning of many different operational challenges.

#### Keeping live and local states in sync

A method of configuring Kubernetes resources that we would encourage is to maintain their configuration
in source control for teams to edit and share, which also allows the source control repository to become
the source of truth. The configuration defined in source control (referred to as the local state) is then
created by applying them to the Kubernetes environment, and the resources become live or enter what
can be called a live state. This sounds simple enough, but what happens when developers need to make
changes to their resources? The proper answer would be to modify the files in source control and apply
the changes to synchronize the local state to the live state. However, this isn’t what always ends up
happening. It is often simpler, in the short term, to modify the live resource in place with kubectl
edit or kubectl patch and completely skip over modifying the local files. This results in state
inconsistency between local and live states and is an act that makes scaling on Kubernetes difficult.

#### Application life cycles are hard to manage

Life cycle management is a loaded term, but in this context, we’ll refer to it as the concept of installing,
upgrading, and rolling back applications. In the Kubernetes world, an installation would include API
resources for deploying and configuring an application. The initial installation would create what we
refer to here as version 1 of an application.
An upgrade, then, can be thought of as a modification to one or many of those Kubernetes resources.
Each batch of edits can be thought of as a single upgrade. A developer could modify a single service
resource, which would bump the version number to version 2. The developer could then modify a
deployment, a configmap, and a service at the same time, bumping the version count to version 3.
As newer versions of an application continue to be rolled out onto Kubernetes, it becomes more difficult
to keep track of changes that have occurred across relevant API resources. Kubernetes, in most cases,
does not have an inherent way of keeping a history of changes. While this makes upgrades harder to
keep track of, it also makes restoring a prior version of an application much more difficult. Say, for
example, a developer previously made an incorrect edit on a particular resource. How would a team
know where to roll back to? The n-1 case is particularly easy to work out, as that is the most recent
version. What happens, however, if the latest stable release was five versions ago? Teams often end
up scrambling to resolve issues because they cannot quickly identify the latest stable configuration
that worked previously.

#### Resource files are static

This is a challenge that primarily affects the declarative configuration style of applying YAML resources.
Part of the difficulty in following a declarative approach is that Kubernetes resource files are not
natively designed to be parameterized. Resource files are largely designed to be written out in full
before being applied, and the contents remain the source of truth (SOT) until the file is modified.
When dealing with Kubernetes, this can be a frustrating reality. Some API resources can be lengthy,
containing many different customizable fields, and it can be quite cumbersome to write and configure
YAML resources in full.

Static files lend themselves to becoming boilerplate. Boilerplate represents text or code that remains
largely consistent in different but similar contexts. This becomes an issue if developers manage multiple
different applications, where they could potentially manage multiple different deployment resources,
multiple different services, and so on. In comparing the different applications’ resource files, you may
find large numbers of similar YAML configurations between them.

### Simplified Introduction to Helm and Package Managers

Over time, deploying applications on Kubernetes proved challenging due to the complexity of managing Kubernetes resources. To address these challenges, Helm, an open-source Kubernetes package manager, was developed. Helm simplifies application deployment on Kubernetes in a manner similar to how traditional package managers work on operating systems.

#### How Package Managers Work

Package managers like `dnf` on Fedora streamline the process of installing, upgrading, reverting, and removing software. They manage packages that include both the software and its dependencies. For example, to install the `htop` system monitor on Fedora, you’d simply run:

```bash
dnf install htop --assumeyes
```

This command instructs `dnf` to find `htop` in the Fedora repository and handle all necessary dependencies. Package managers also allow for easy upgrades, downgrades, and removal of software, making software management straightforward.

#### Helm: The Kubernetes Package Manager

Helm operates similarly to traditional package managers but is tailored for Kubernetes. Instead of RPM packages, Helm uses charts, which are collections of Kubernetes resource files required to deploy applications. These charts can be found in repositories, just as software packages can be found in OS repositories.

For example, deploying Redis on Kubernetes using Helm is as simple as running:

```bash
helm install redis bitnami/redis --namespace=redis
```

This command installs the Redis chart from the Bitnami repository into the `redis` namespace. Helm also offers commands for upgrading (`helm upgrade`), rolling back (`helm rollback`), and uninstalling (`helm uninstall`) applications, mirroring the functionality of traditional package managers like `dnf`.

#### Benefits of Using Helm

Helm offers several advantages:

1. **Simplifies Kubernetes Resource Management**: Helm abstracts the complexity of Kubernetes, enabling developers to deploy applications like WordPress by simply searching for existing charts, without needing extensive Kubernetes knowledge.

2. **Maintains Revision History**: Helm keeps a history of application revisions, allowing users to roll back to any previous state with ease.

3. **Dynamic Resource Configuration**: Helm introduces values and templates, enabling parameterization of Kubernetes resources. This reduces boilerplate code and makes managing applications more flexible.

4. **Synchronizes Local and Live States**: Helm manages the intricate details of Kubernetes resources, allowing users to focus on local configuration while Helm handles updates and synchronization.

5. **Automated Lifecycle Management**: Helm supports lifecycle hooks for automating tasks like backups, rollbacks, and environment validation, simplifying the management of Kubernetes applications.

### Summary

In summary, Helm is a powerful tool that simplifies the management of Kubernetes applications, offering an experience similar to traditional package managers. By using Helm, teams can efficiently manage the full lifecycle of applications, from installation to rollback, with minimal Kubernetes expertise required.

Next, we’ll explore installing Helm and setting up an environment to work with Helm charts.

