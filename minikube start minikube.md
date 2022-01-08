# minikube start | minikube
minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

All you need is Docker (or similarly compatible) container or a Virtual Machine environment, and Kubernetes is a single command away: `minikube start`

## What you’ll need

-   2 CPUs or more
-   2GB of free memory
-   20GB of free disk space
-   Internet connection
-   Container or virtual machine manager, such as: [Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/), [Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/), [Hyper-V](https://minikube.sigs.k8s.io/docs/drivers/hyperv/), [KVM](https://minikube.sigs.k8s.io/docs/drivers/kvm2/), [Parallels](https://minikube.sigs.k8s.io/docs/drivers/parallels/), [Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/), [VirtualBox](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/), or [VMware Fusion/Workstation](https://minikube.sigs.k8s.io/docs/drivers/vmware/)

## **1**Installation

Click on the buttons that describe your target platform. For other architectures, see [the release page](https://github.com/kubernetes/minikube/releases/latest) for a complete list of minikube binaries.

To install the latest minikube **stable** release on **x86-64** **Linux** using **binary download**:

To install the latest minikube **beta** release on **x86-64** **Linux** using **binary download**:

To install the latest minikube **stable** release on **x86-64** **Linux** using **Debian package**:

To install the latest minikube **beta** release on **x86-64** **Linux** using **Debian package**:

To install the latest minikube **stable** release on **x86-64** **Linux** using **RPM package**:

To install the latest minikube **beta** release on **x86-64** **Linux** using **RPM package**:

To install the latest minikube **stable** release on **ARM64** **Linux** using **binary download**:

To install the latest minikube **beta** release on **ARM64** **Linux** using **binary download**:

To install the latest minikube **stable** release on **ARM64** **Linux** using **Debian package**:

To install the latest minikube **beta** release on **ARM64** **Linux** using **Debian package**:

To install the latest minikube **stable** release on **ARM64** **Linux** using **RPM package**:

To install the latest minikube **beta** release on **ARM64** **Linux** using **RPM package**:

To install the latest minikube **stable** release on **ppc64** **Linux** using **binary download**:

To install the latest minikube **beta** release on **ppc64** **Linux** using **binary download**:

To install the latest minikube **stable** release on **ppc64** **Linux** using **Debian package**:

To install the latest minikube **beta** release on **ppc64** **Linux** using **Debian package**:

To install the latest minikube **stable** release on **ppc64** **Linux** using **RPM package**:

To install the latest minikube **beta** release on **ppc64** **Linux** using **RPM package**:

To install the latest minikube **stable** release on **S390x** **Linux** using **binary download**:

To install the latest minikube **beta** release on **S390x** **Linux** using **binary download**:

To install the latest minikube **stable** release on **S390x** **Linux** using **Debian package**:

To install the latest minikube **beta** release on **S390x** **Linux** using **Debian package**:

To install the latest minikube **stable** release on **S390x** **Linux** using **RPM package**:

To install the latest minikube **beta** release on **S390x** **Linux** using **RPM package**:

To install the latest minikube **stable** release on **ARMv7** **Linux** using **binary download**:

To install the latest minikube **beta** release on **ARMv7** **Linux** using **binary download**:

To install the latest minikube **stable** release on **ARMv7** **Linux** using **Debian package**:

To install the latest minikube **beta** release on **ARMv7** **Linux** using **Debian package**:

To install the latest minikube **stable** release on **ARMv7** **Linux** using **RPM package**:

To install the latest minikube **beta** release on **ARMv7** **Linux** using **RPM package**:

To install the latest minikube **stable** release on **x86-64** **macOS** using **Homebrew**:

If the [Homebrew Package Manager](https://brew.sh/) is installed:

If `which minikube` fails after installation via brew, you may have to remove the old minikube links and link the newly installed binary:

To install the latest minikube **stable** release on **x86-64** **macOS** using **binary download**:

To install the latest minikube **beta** release on **x86-64** **macOS** using **binary download**:

To install the latest minikube **stable** release on **ARM64** **macOS** using **binary download**:

To install the latest minikube **stable** release on **ARM64** **macOS** using **Homebrew**:

If the [Homebrew Package Manager](https://brew.sh/) is installed:

If `which minikube` fails after installation via brew, you may have to remove the old minikube links and link the newly installed binary:

To install the latest minikube **beta** release on **ARM64** **macOS** using **binary download**:

To install the latest minikube **stable** release on **x86-64** **Windows** using **Windows Package Manager**:

If the [Windows Package Manager](https://docs.microsoft.com/en-us/windows/package-manager/) is installed, use the following command to install minikube:

To install the latest minikube **stable** release on **x86-64** **Windows** using **Chocolatey**:

If the [Chocolatey Package Manager](https://chocolatey.org/) is installed, use the following command:

To install the latest minikube **stable** release on **x86-64** **Windows** using **.exe download**:

1.  Download the [latest release](https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe).

    Or if using `PowerShell`, use this command:
2.  Add the binary in to your `PATH`.

    _Make sure to run PowerShell as Administrator._

    _If you used a CLI to perform the installation, you will need to close that CLI and open a new one before proceeding._

To install the latest minikube **beta** release on **x86-64** **Windows** using **.exe download**:

2.  Add the binary in to your `PATH`.

    _Make sure to run PowerShell as Administrator._

    _If you used a CLI to perform the installation, you will need to close that CLI and open a new one before proceeding._

## **2**Start your cluster

From a terminal with administrator access (but not logged in as root), run:

If minikube fails to start, see the [drivers page](https://minikube.sigs.k8s.io/docs/drivers/) for help setting up a compatible container or virtual-machine manager.

## **3**Interact with your cluster

If you already have kubectl installed, you can now use it to access your shiny new cluster:

Alternatively, minikube can download the appropriate version of kubectl and you should be able to use it like this:

You can also make your life easier by adding the following to your shell config:

Initially, some services such as the storage-provisioner, may not yet be in a Running state. This is a normal condition during cluster bring-up, and will resolve itself momentarily. For additional insight into your cluster state, minikube bundles the Kubernetes Dashboard, allowing you to get easily acclimated to your new environment:

## **4**Deploy applications

Create a sample deployment and expose it on port 8080:

It may take a moment, but your deployment will soon show up when you run:

The easiest way to access this service is to let minikube launch a web browser for you:

Alternatively, use kubectl to forward the port:

Tada! Your application is now available at [http://localhost:7080/](http://localhost:7080/).

You should be able to see the request metadata from nginx such as the `CLIENT VALUES`, `SERVER VALUES`, `HEADERS RECEIVED` and the `BODY` in the application output. Try changing the path of the request and observe the changes in the `CLIENT VALUES`. Similarly, you can do a POST request to the same and observe the body show up in `BODY` section of the output.

### LoadBalancer deployments

To access a LoadBalancer deployment, use the “minikube tunnel” command. Here is an example deployment:

In another window, start the tunnel to create a routable IP for the ‘balanced’ deployment:

To find the routable IP, run this command and examine the `EXTERNAL-IP` column:

Your deployment is now available at <EXTERNAL-IP>:8080

## **5**Manage your cluster

Pause Kubernetes without impacting deployed applications:

Unpause a paused instance:

Halt the cluster:

Increase the default memory limit (requires a restart):

Browse the catalog of easily installed Kubernetes services:

Create a second cluster running an older Kubernetes release:

Delete all of the minikube clusters:

## Take the next step

-   [The minikube handbook](https://minikube.sigs.k8s.io/docs/handbook/)
-   [Community-contributed tutorials](https://minikube.sigs.k8s.io/docs/tutorials/)
-   [minikube command reference](https://minikube.sigs.k8s.io/docs/commands/)
-   [Contributors guide](https://minikube.sigs.k8s.io/docs/contrib/)
-   Take our [fast 5-question survey](https://forms.gle/Gg3hG5ZySw8c1C24A) to share your thoughts 🙏 
    [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
