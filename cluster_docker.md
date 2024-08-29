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

##### I was researching for better manage of tdarr node is better use docker. 

## 3. Install Docker

#### Install docker
```command
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

#### Enable and run docker
```command
sudo systemctl enable docker
sudo systemctl start docker
```

## 4. Install Tdarr using docker

#### Set up Tdarr Docker Compose

###### create a new directory for Tdarr and docker compose files 

```command
mkdir -p ~/tdarr && cd ~/tdarr
nano docker-compose.yml
```

#### Docker compose configuration.

#### Node1: 

```text
version: '3'
services:
  tdarr_server:
    image: haveagitgat/tdarr:latest
    container_name: tdarr_server
    volumes:
      - /path/to/config:/app/configs
      - /path/to/logs:/app/logs
      - /path/to/media:/media
    ports:
      - "8265:8265"
    environment:
      - serverIP=0.0.0.0
    restart: always

  tdarr_node:
    image: haveagitgat/tdarr_node:latest
    container_name: tdarr_node
    volumes:
      - /path/to/config:/app/configs
      - /path/to/logs:/app/logs
      - /path/to/media:/media
    environment:
      - serverIP=<head_node_ip>
      - nodeID=tdarr_node_1
    restart: always
```

#### Node2:

```text
version: '3'
services:
  tdarr_node:
    image: haveagitgat/tdarr_node:latest
    container_name: tdarr_node
    volumes:
      - /path/to/config:/app/configs
      - /path/to/logs:/app/logs
      - /path/to/media:/media
    environment:
      - serverIP=<head_node_ip>
      - nodeID=tdarr_node_2
    restart: always
```

#### Start docker on both nodes

```command
cd ~/tdarr
sudo docker-compose up -d
```

###### Note: You must set the node compute from the web UI and verify if the main node is detected. 

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
