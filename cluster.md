# Create cluster for transcoding

## 1. Configure ssh

#### Generate ssh key
```command
ssh-keygen -t rsa
```
#### Copy ssh key

```command
ssh-copy-id user@servidor2
```

## 2. (optional) Configure Firewall to allow only necessary ports.

 - *using iptables or ufw to can allow ssh ports between nodes*
 - *allow ports for tdarr node*

Tdarr node use the port 8256 

## 3. Install Tdarr (Manually)

#### Follow instructions according Tdarr Website 

 - Here [instructions...](https://docs.tdarr.io/docs/installation/windows-linux-macos)

## 5. Install GlusterFS (this for shared storage)

#### Install GlusterFS 
```command
sudo yum install -y centos-release-gluster
sudo yum install -y glusterfs-server
```

#### Set up GlusterFS Volume
- replace /data with you folder to share
- you must set up on main node (Node1)

```command
sudo gluster volume create gvol0 replica 2 node1:/data node2:/data
sudo gluster volume start gvol0
```

#### Mount GlusterFS on both nodes

```command
sudo mount -t glusterfs node1:/gvol0 /path/to/media
```
###### Note: Add this to /etc/fstab for automatic mounting on boot.

```command
echo "node1:/gvol0 /path/to/media glusterfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab
```

## 6. Configure Cluster Management Tool (Optional)

##### It's not necessary, but can help to check and review resources of cluster

```command
sudo yum install pacemaker corosync pcs -y
```

#### Set up cluster auth

##### this for secure connection between both nodes 

```command
sudo passwd hacluster
```

#### Authenticate both nodes

```command
sudo pcs cluster auth node1 node2
```

#### Create HA Cluster

```command
sudo pcs cluster setup --name transcoding-cluster node1 node2
```
#### Start and Enable

```command
sudo pcs cluster start --all
sudo pcs cluster enable --all
```
