# AWS-PrivateVPC-KubernetesCluster
Kubernetes Cluster in AWS Private VPC

Goals -
  - A VPC with two subnets - Public Subnet and Private Subnet
  - Kubernetes Cluster deployed in Private Subnet
  - Cluster will access internet only through NAT Instance hosted in Public Subnet
  - We will login and talk to our cluster using Bastion Host hosted in Public Subnet.

Steps:
  - Create VPC
  - Create two subnets - give tag private_projectname and public_projectname
  - Create an internet gateway
  - Create two route table - Public and Private RouteTable
  - Associate public route table to public subnet and private route table to private subnet
  - Associate public route table to internet gateway, to open communication with open internet
  - Launch three ubuntu instances in Custom VPC and within Private Subnet. Two instances for worker node and one instance for master node
  - Master node instance must have atleast 2vcpu and 4GB ram. Open security group for ports like ssh and anywhere from world or bastion ip.
  - Launch one instance in public subnet and enable the public ip association. By default in custom vpc public ip option is disabled.
  - Create one nat gateway in public subnet and associate elastic ip.
  - Associate Nat Gateway to private route table, so kubernetes cluster can talk to internet.
  
Next Steps:
  - Login/ssh in bastion host and upload key file (for login in resources hosted in private subnet)
    - scp -i ~/Downloads/KubernetesDemo_VPC.pem KubernetesDemo_VPC.pem ubuntu@ec2-52-37-234-217.us-west-2.compute.amazonaws.com:/home/ubuntu
  - Change the file permissions for the key file
	  - chmod 400 KubernetesDemo_VPC.pem
	- ssh in master node and two worker nodes
	- Run following commands:
	  - sudo apt-get update
	  - sudo apt-get install apt-transport-https  (https package is needed for intra cluster communication)
	  - sudo apt-get install docker.io -y
	- Go to root priviledge sudo su and run
	  - systemctl start docker
	  - systemctl enable docker
  - We will download and add one gpg key file in all nodes
	  - sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
  - Created one file in all nodes and write command in file to download kubernetes xenial.
	  - nano /etc/apt/sources.list.d/kubernetes.list
	  - deb http://apt.kubernetes.io/ kubernetes-xenial main
	- For Bootstraping master node
	  - kubeadm init
	
  Note : Note- Here if you face kubelet not starting error. Then,Create this file "daemon.json" in the directory "/etc/docker" and add the following
  {
  "exec-opts": ["native.cgroupdriver=systemd"]
  }

  Restart your docker service: systemctl restart docker
  Reset kubeadm initializations: kubeadm reset
  Initialize your network: kubeadm init 

  If docker is not starting using systemctl restart docker and giving masked docker unit error then -
	  - sudo systemctl unmask docker
	  - sudo systemctl start docker
    
 Finally Last step:
    - After bootstaping master node, we will get one kubeadm join command. We need to paste the command in all worker nodes.
    - To start using your cluster, you need to run the following as a regular user in master node:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

   - Copy kubeadm join command and paste it in all worker nodes.
	   - kubeadm join 10.0.2.37:6443 --token 0zhy3t.jndir1ltpeytws0t \
        --discovery-token-ca-cert-hash sha256:4a046c9245fe82b5e2aa564e470ce80f35439e223ae80671b9c5e74965fcd570
	 
