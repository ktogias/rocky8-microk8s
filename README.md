# rocky8-microk8s

This is an [Ansible](https://github.com/ansible/ansible) based GitOps project for setting up a [Microk8s](https://microk8s.io/) cluster on nodes running [Rocky8](https://rockylinux.org/).

It is meant to be used for development, testing and learning purposes. It does not provide storage replication and other High Availability features, so it is not suitable for hosting production payloads.

## What you will need

1. SSH access to one or more vms (or physical machines) with a minimal install of Rocky8 (Playbooks should work on any RHEL8 based distro). See https://docs.rockylinux.org/guides/installation/ for Rocky Linux installation instructions. Your account on the nodes must be able to sudo or be root.
2. Your workstation running a linux OS with Ansible installed. (I have not tried to run the playbooks from a windows or macos PC). You must be able to SSH to your cluster nodes from your workstation. 


## Setting up the cluster

1. Copy your SSH public key to each of your nodes, so ansible can ssh without asking for password:  
   ```ssh-copy-id <node-ip>``` 
2. Git clone this repo (e.g. ```git clone https://github.com/ktogias/rocky8-microk8s.git```)
3. Open your terminal in rocky8-microk8s folder (e.g. ```cd rocky8-microk8s```)
4. Run ansible-host-configuration.sh scrit. This will install the reqired Ansible extensions:  
  ```./ansible-host-configuration.sh```
5. Copy ```hosts.ini.dist``` to ```hosts.ini``` and edit it to match your node machines info:   
   ```cp hosts.ini.dist hosts.ini```  
   The ```[all]``` group has info about all the nodes. Fill the ssh port (ansible_port), the ip (ansible_host) and the username (ansible_user) you use to SSH to your nodes.  
   The ```[init]``` group should conatin the name of the initial node from where you are going to initiate the cluster. It will be a kuberentes master (control plane) for the cluster.  
   The ```[masters]``` group should contain all the nodes that will have a kuberentes master (control plane) role.  
   The ```[workers]``` group should contain all the nodes that will have kuberentes worker role.   
6. Copy ```vars.yaml.dist``` to ```vars.yaml``` and edit it to match your configuration. See file comments for more info.  
   ```cp vars.yaml.dist vars.yaml```  
7. Run the ```hosts_setup``` palybook:  
   ```ansible-playbook -i hosts.ini -e @vars.yaml hosts_setup.yaml -K```  
   This will update the OS, install and configure your nodes' software  
8. If you have more than on node and want them to form one cluster run the ```add_nodes``` playbook:  
   ```ansible-playbook -i hosts.ini -e @vars.yaml add_nodes.yaml -K```  
   This will make each node in the ```[workers]``` group join the cluster initiated on the node in ```[init]``` group.  
   
That's it! Now you have a running Microk8s Kuberentes Cluster!!!  

## Enabling Microk8s utilities
   
To enable basic Microk8s utilities such as ingress, dns, dashboard, storage, registry and helm3 run  ```deploy_utils``` playbook:  
   ```ansible-playbook -i hosts.ini -e @vars.yaml deploy_utils.yaml -K```  
   
To access kuberentes dashboard open your browser to https://```dashboardIngressHost```/dashboard/ (see ```vars.yaml```) and use the token displayed by ```deploy_utils``` playbook.

## Deploying GitLab

Now that you have a kuberentes cluster you are ready to start developing the projects that you will host or test on it.  
Something is missing though. You need a collaboration platform for you and your DevOps team.   
So let's deploy a self hosted [GitLab](https://about.gitlab.com/) instance on our cluster!  

* Please read [GitLab Helm Chart Requirements](https://docs.gitlab.com/charts/installation/#requirements).  
If your cluster is too small in terms of CPU cores and RAM for GitLab the deployment will fail.

1. Copy ```gitlab_vars.yaml.dist``` to ```gitlab_vars.yaml``` and edit it to match your configuration. See file comments for more info.  
   ```cp gitlab_vars.yaml.dist gitlab_vars.yaml```  
2. Run the ```deploy_gitlab``` palybook with ```gitlab_vars```:  
   ```ansible-playbook -i hosts.ini -e @gitlab_vars.yaml deploy_gitlab.yaml -K```  

Wait for the deployment to complete (it may take several minutes) and then open your browser to https://gitlab.```domain``` (see ```gitlab_vars.yaml```) and login using the root account and root password (see ```rootPassword``` in ```gitlab_vars.yaml```).  


## Known issues
1. When having multiple nodes pods may fail with ```Permission denied``` error for PersistentVolumes. Run ```chmod ugo+rwX /var/snap/microk8s/common/default-storage/*``` on each node to fix permissions.
2. When using self-signed certificates Gitlab runner pod is failling with error message: ```Post https://gitlab.<your-host-name>/api/v4/runners: x509: certificate signed by unknown authority```.  Edit ```gitlab-tls-secret``` in ```gitlab``` namespace and copy ```ca.crt``` to ```gitlab.<domain>.crt```
