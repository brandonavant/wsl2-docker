# Installing Docker Inside WSL 2

This repository simply provides instructions on how to quickly install Docker inside WSL 2, without the need for Docker Desktop. Please read this document from top to bottom to ensure that you meet each step's prerequisites.

## Install WSL 2

Prior to installing Docker inside WSL 2, we must first install WSL 2. To achieve this, please follow the [official instructions](https://docs.microsoft.com/en-us/windows/wsl/install) provided by Microsoft.

## Install Ubuntu

Once WSL 2 is ready to go, install Ubuntu inside your new WSL 2 instance. Use the [official instructions](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview) from Ubuntu to achieve this.

Once the installation is complete, we must ensure that our repos are up-to-date by running the following commands:

```bash
sudo apt update
sudo apt upgrade
```

We are now ready to install Docker inside Windows Subsystem for Linux v2.

## Install Docker

I originally considered typing out all of the instructions in this repo for installing Docker; however, I determined that the [official instructions](https://docs.docker.com/engine/install/ubuntu/#installation-methods) from Docker are sufficient, with some minor tweaks once the official installation is complete.

That being said, please look [here](https://docs.docker.com/engine/install/ubuntu/#installation-methods) and follow the official Docker for Ubuntu instructions.

Once you've installed docker, attempt to start the service:

```bash
sudo service docker start
```

and then check the status

```bash
sudo service docker status
```

> NOTE: I noticed that if you run both `start` and `status` commands in quick succession (e.g. by using the `&&` operator), you might get a false positive showing that the status is running when it's not. If you run the commands using the `&&` operator, follow up by checking the status once again.

There is a good chance that you will receive a status indicating that Docker is _not_ running -- i.e. you might see this status:

```bash
* Docker is not running
```

This is commonly caused by an issue with using the newest IP table configuration. We must switch to using legacy using the following command:

```bash
sudo update-alternatives --config iptables
```

You will be prompted to make a selection:

```bash
There are 2 choices for the alternative iptables (providing /usr/sbin/iptables).

  Selection    Path                       Priority   Status
------------------------------------------------------------
* 0            /usr/sbin/iptables-nft      20        auto mode
  1            /usr/sbin/iptables-legacy   10        manual mode
  2            /usr/sbin/iptables-nft      20        manual mode
```

Choose selection `1` for the legacy IP tables. Next, try starting the service and checking its status once again:

```bash
sudo service docker start
sudo service docker status
```

You should now receive a _Running_ status:

```bash
* Starting Docker: docker                                                            [ OK ]
* Docker is running
```

We are not done yet! We must now ensure that we automatically start the service each time our WSL 2 instance starts. To do this, we need to modify a few files.

First, run we must modify the sudoers configuration file:

```bash
sudo visudo
```

Add the following to the bottom of the file:

```bash
# Allow user to utse service without sudo
<your-username> ALL=(ALL) NOPASSWD: /usr/sbin/service
```

We need to also ensure that we can run docker without needing `sudo`. To do this, run the following command:

```bash
sudo usermod -aG docker $USER && newgrp docker
```

Next, modify `~/.bashrc` and add the following lines to the bottom of the file:

```bash
STATUS="$(sudo service docker status)"
if [ "${STATUS}" = " * Docker is not running" ]; then
  sudo service docker start > /dev/null 2>&1
fi
```

Finally, open a PowerShell session and run this command to shutdown WSL 2 (replacing the name of the Ubuntu instance accordingly):

```powershell
wsl --terminate Ubuntu-22.04
```

Open a new Ubuntu terminal session and check the status to make sure Docker is running

```bash
sudo service docker status
```

## Install Minikube

To install Minikube (as a local single-node Kubernetes environment), follow the instructions [here](https://minikube.sigs.k8s.io/docs/start/).

> NOTE: It will likely be necessary to run the following command prior to running `minikube start` in order to set the default driver as _Docker_:

```bash
minikube config set driver docker
```
