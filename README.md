# Set up a Minikube development environment
## Install VMware Workstation Pro (optional)
This option is supposedly the most compatible with container deployment to vSphere. The Workstation Pro client can be downloaded for free. You will need a Broadcom support portal account, but you can create one for free.

### Windows:
[Download](https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Workstation%20Pro&displayGroup=VMware%20Workstation%20Pro%2017.0%20for%20Windows&release=17.6.4&os=&servicePk=&language=EN&freeDownloads=true)

The Linux client can also be installed

Once installed, ensure that `vmrun` is available from the command line by verifying that `C:\Program Files (x86)\VMware\VMware Workstation` exists in system PATH.

> NOTE: I had `C:\Program Files (x86)\VMware\VMware Workstation\bin`, but not `C:\Program Files (x86)\VMware\VMware Workstation` and had to add it manually.

### Ubuntu (and variants):
[Download](https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Workstation%20Pro&displayGroup=VMware%20Workstation%20Pro%2017.0%20for%20Linux&release=17.6.4&os=&servicePk=&language=EN&freeDownloads=true)

Install prerequisites first by running:
```
sudo apt update
sudo apt install -y build-essential -y
```

To install VMware Workstation Pro, first navigate to the folder it was downloaded to. Then run this command, substituting the name of your specific file, which will be different as newer versions are released.
```
sudo bash VMware-Workstation-Full-17.6.4-24832109.x86_64.bundle
```

Install additional required Kernel modules by running:
```
sudo vmware-modconfig --console --install-all
```

## Install Docker (optional)
You must install either VMware Workstation Pro or Docker. You can install both if desired.

### Windows 11:
```
winget install Docker.DockerCLI
```

### Ubuntu (and variants):
Install Docker's apt repository:
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Arch Linux (and variants):
```
sudo pacman -Syu
sudo pacman -S docker
```
NOTE: By default, Docker runs as a system service. As a best practice, Docker can be run rootless. This requires additional set up.

Install the docker-rootless package from the Arch User Repository:
```
sudo paru -S docker-rootless
```
OR
```
sudo yay -S docker-rootless
```
You will need an AUR helper to install all components. Two common helpers are Yay and Paru. Paru is my preference. It can be installed by following the installation instructions on it's [Github](https://github.com/Morganamilo/paru).

## Install Kubectl
kubectl is a command-line tool used to interact with Kubernetes clusters. It acts as a bridge between users and the Kubernetes control plane, allowing users to manage and operate Kubernetes resources such as pods, services, deployments, and more.

### Windows 11:
```
winget install Kubernetes.kubectl
```
Verify kubectl is installed by running this command.
```
kubectl version --client
```
### Linux:
Ubuntu (and variants):
> NOTE: The classic tag here is extremely important. This ensures that kubectl is installed outside of a sandbox.
```
sudo snap install kubectl --classic
```

Arch Linux (and variants):
```
sudo pacman -Syu
sudo pacman -S kubectl
```

## Install Minikube
Minikube is a lightweight Kubernetes implementation that creates a single-node Kubernetes cluster on your local machine. It is designed to simplify local Kubernetes development, making it accessible for developers to learn and test Kubernetes-based applications without needing a full multi-node cluster or cloud resources. Minikube is available for various operating systems, including Linux, macOS, and Windows.

### Windows 11:
```
winget install Kubernetes.minikube
```
Verify that minikube was install successfully by running:
```
minikube version
```

### Linux:
Ubuntu (and variants):
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```
Verify that minikube was install successfully by running:
```
minikube version
```

Arch Linux (and variants):
```
sudo pacman -Syu
sudo pacman -S minikube
```
Verify that minikube was install successfully by running:
```
minikube version
```

## Start Minikube
If you installed VMware Workstation Pro, you can start minikube by running:
```
minikube start --driver=vmware
```
If you installed Docker, you can run:
```
minikube start --driver=docker
```
You should now successfully have a minikube development environment up and running.

# Usage Examples
## Drupal
### Install Helm:
#### Windows
Helm is the best way to find, share, and use software built for Kubernetes. Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

Learn more here: https://www.freecodecamp.org/news/what-is-a-helm-chart-tutorial-for-kubernetes-beginners/

Windows:
```
winget install Helm.Helm
```
Ubuntu (and variants):
> NOTE: The classic tag here is extremely important. This ensures that helm is installed outside of a sandbox.
```
sudo snap install helm --classic
```
Arch Linux (and variants):
```
sudo pacman -S helm
```
### Install Drupal using Helm
First add the Bitnami repository to Helm:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
Next Install the Drupal Helm chart. Replace `my-drupal` with your preferred release name. Set secure passwords for Drupal admin and the database root (avoid defaults in production-like setups).
```
helm install my-drupal oci://registry-1.docker.io/bitnamicharts/drupal --set drupalUsername=admin --set drupalPassword=your-secure-password --set mariadb.auth.rootPassword=your-db-root-password --set mariadb.auth.password=your-db-password
```
It will take several minutes to deploy. You can monitor the status of the deployment by running the following command. Substitute the name you used for your drupal instance.
```
kubectl get svc --namespace default -w my-drupal
```
Once installed you can get the service IP address by running the following command. Substitute my-drupal with the name you used for your drupal instance.
```
minikube service my-drupal --url
```
Copy and paste the first URL into a browser and you should see the default Drupal homepage.
