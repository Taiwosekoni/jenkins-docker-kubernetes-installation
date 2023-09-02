# jenkins-docker-kubernetes-installation
sudo apt update -y
sudo apt install docker.io -y 
sudo systemctl start docker
sudo systemctl enable docker.service
sudo usermod -aG docker ubuntu
sudo systemctl restart docker.service
echo "ubuntu  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ubuntu
sudo apt install openjdk-11-jdk -y

#install start and configure jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo usermod -aG docker jenkins
sudo systemctl restart docker.service
echo "jenkins  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d./jenkins

#install kops software on ubuntu server
sudo apt install wget -y
sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
sudo chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

#install AWSCLI by running the command bellow
sudo apt install awscli -y
#create an IAM-Role and attach the following permissions to it 
AmazonEC2FullAccess 
AmazonS3FullAccess
IAMFullAccess 
AmazonVPCFullAccess

 #create an S3 bucket
Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.
aws s3 mb s3://class30kops
aws s3 ls # to verify

#create an S3 bucket
Expose environment variable:
# Add env variables in bashrc

   vi .bashrc
# Give Unique Name And S3 Bucket which you created.
export NAME=class30.k8s.local
export KOPS_STATE_STORE=s3://class30kops

  source .bashrc  
#create sshkey before creating cluster
ssh-keygen


 #Install kubectl kubernetes client if it is not already installed
sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
 sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl

#CREATE K*S CKUSTER ON DEFINITION S3 BUCKECT
kops create cluster --zones us-west-2a --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}
# copy the sshkey into your cluster to be able to access your kubernetes node from the kops server

#COPY THE SSHKEY BY RUNNING THE COMMAND BELLOW
kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub

#UPDATE THE CLUSTER
kops update cluster ${NAME} --yes

 Export the kubeconfig file to manage your kubernetes cluster from a remote server. For this demo, Our remote server shall be our kops server 
 kops export kubecfg $NAME --admin

#XECUTE THE BELLOW COMMAND TO SSH INTO THE FROM JENKINS CLI (CHANGE THE IPADDRESS BEFORE RUNNING) 
 ssh -i ~/.ssh/id_rsa ubuntu@ec2-54-91-150-215.compute-1.amazonaws.com 
