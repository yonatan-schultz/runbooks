##Configuring Kubernetes Cluster

This runbook will help you set up a Kubernetes cluster on Raspberry Pi using Hypiot and Docker-Multinode.

###Installing Hypriot

Instead of Raspbian we'll use Hypriot 1.0.0 which already has Docker 1.12.1 installed. Install this on your SD card in the usual way:

```
diskutil list (locate SD card, eg /dev/disk2)
diskutil unmountDisk /PATH/TO/SDCARD
sudo dd bs=1m if=hypriotos-rpi-v1.0.0.img of=/PATH/TO/SDCARD
```

Once you've written out your SD cards, insert them into the Raspberry Pi and power on the devices. The default usename is `pirate` and password is `hypriot`.

We'll need to do some basic securing of the host. First, run `passwd` and change the password of the pirate user. Install puppet and run the agent. The base puppet profile will finish securing the host (securing ssh, adding user, authorized_keys).

```
sudo apt-get update
sudo apt-get install puppet
sudo puppet agent --enable
sudo puppet agent -t
```

Lastly, update the hostname by editing `/boot/cmdline.txt`.

Now we're ready to configure the Kubernetes master.

###Configuring the Master

We'll be using Kubernetes' docker-multinode which will automate the setup of the master (and workers) for us.

```
sudo su -
git clone https://github.com/kubernetes/kube-deploy
cd kube-deploy/docker-multinode
./master.sh
```

The output should be similar to:

```
+++ [0903 17:37:53] K8S_VERSION is set to: v1.3.6
+++ [0903 17:37:53] ETCD_VERSION is set to: 2.2.5
+++ [0903 17:37:53] FLANNEL_VERSION is set to: v0.6.1
+++ [0903 17:37:53] FLANNEL_IPMASQ is set to: true
+++ [0903 17:37:53] FLANNEL_NETWORK is set to: 10.1.0.0/16
+++ [0903 17:37:53] FLANNEL_BACKEND is set to: udp
+++ [0903 17:37:53] RESTART_POLICY is set to: unless-stopped
+++ [0903 17:37:53] MASTER_IP is set to: localhost
+++ [0903 17:37:53] ARCH is set to: arm
+++ [0903 17:37:53] IP_ADDRESS is set to: 192.168.1.219
+++ [0903 17:37:53] USE_CNI is set to: false
+++ [0903 17:37:53] --------------------------------------------
+++ [0903 17:37:54] Killing docker bootstrap...
+++ [0903 17:38:46] Killing all kubernetes containers...
Do you want to clean /var/lib/kubelet? [Y/n]
+++ [0903 17:38:57] Launching docker bootstrap...
+++ [0903 17:38:59] Launching etcd...
3415e76f483bdd0665fca69ebe1f95349c2c23d3d301c51cbbd353f888f27f6d
+++ [0903 17:39:04] Launching flannel...
{"action":"set","node":{"key":"/coreos.com/network/config","value":"{ \"Network\": \"10.1.0.0/16\", \"Backend\": {\"Type\": \"udp\"}}","modifiedIndex":4,"createdIndex":4}}
65d82372754f2a0132f2118ef3f68f133197ff044548321e87b8f16ac7055c3f
+++ [0903 17:39:06] FLANNEL_SUBNET is set to: 10.1.75.1/24
+++ [0903 17:39:06] FLANNEL_MTU is set to: 1472
+++ [0903 17:39:06] Restarting main docker daemon...
+++ [0903 17:39:11] Restarted docker with the new flannel settings
+++ [0903 17:39:11] Launching Kubernetes master components...
fb3912bb67333c436221034476b6404c9d8895c2ce286204b5890b25d9432c01
+++ [0903 17:39:12] Done. It may take about a minute before apiserver is up.
```

Now we need to install `kubectl` in order to interact with our cluster.

```
curl -sSL https://storage.googleapis.com/kubernetes-release/release/v[K8S_VERSION]/bin/linux/amd64/kubectl > /usr/local/bin/kubectl
chmod +x /usr/local/bin/kubectl
```

Now, when you run `kubectl get nodes` you should see your master node:

```
kubectl get nodes
NAME            STATUS    AGE
192.168.1.219   Ready     2m
```

###Configuring the Workers

Configuring workers is almost identical to configuring the master.

```
sudo su -
git clone https://github.com/kubernetes/kube-deploy
cd kube-deploy/docker-multinode
export MASTER_IP=192.168.1.219
./worker.sh
```
Of course, edit the MASTER_IP environment variable to be the IP address of your master.

Once the worker(s) has started, running `kubectl get nodes` on the master again should show the entire cluster up:

```
kubectl get nodes
NAME            STATUS    AGE
192.168.1.219   Ready     8m
192.168.1.220   Ready     6m
192.168.1.221   Ready     1m
```

###Testing the Cluster

The Hypriot group has provided a number of ARM specific Docker images to build upon at `https://hub.docker.com/r/hypriot/rpi-node/`.

We'll run a small nginx test server to verify that the cluster is working:

```
kubectl run busybox --image=hypriot/rpi-busybox-httpd --port=80 --replicas=3
kubectl expose deployment busybox --target-port=80 --type=LoadBalancer --external-ip=MASTER_IP_ADDRESS
```

This will create three copies of the docker image behind a load balancer at the external IP of the master. Now navigating in a web browser to MASTER_IP_ADDRESS you should see:

![images/hypriot.png](images/hypriot.png)


###Sources
https://github.com/kubernetes/kube-deploy/tree/master/docker-multinode
http://blog.hypriot.com/
http://kubernetes.io/docs/user-guide/quick-start/