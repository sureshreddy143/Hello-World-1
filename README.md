ii#Hello-World Java project
CI/CD project using simple hello-world Java Apllication

Create a AWS account
Launch the EC-2 Instances
t2.micro
t3a.medium

#How to setup the each tools in the ec2-servers
-----------------------
      Jenkins
-----------------------
# installation of the jenkins
           https://techviewleo.com/install-jenkins-server-on-amazon-linux/
## * install the git,maven,open-jdk
 * open the jenkins using the specific IP and Port
 * download the suggested plugins or manually you can select the plugins
 
 
 ---------------------------
 Ansible
 ---------------------------
# installation of ansible
 yum install ansible2 python python-pip python-devel -y
#SSH configurations
  #do this in master server as well as jenkins server
    useradd <username>
    passwd <username>
    using visudo command give the sudo previliges for user
    vi /etc/ssh/sshd_config => change passwordauthentication no to yes
    service sshd restart
    switch to the user => sudo su <username>
  
  #only in master server
    ssh-keygen
    ssh-copy-id <username>@<IP-address>
  -----------
   define the private ip of the jenkins server in vi /etc/ansible/hosts
   ansible -m ping all => it will ping then everything configured correctly
  
-------------------
     Docker
--------------------
  yum install docker -y
	usermod -aG docker jenkins 
	usermod -aG docker ec2-user
	service jenkins restart
  
-----------------
    kubernetes
------------------

# Following all commands should e executed on the master as well as nodes
  172.31.40.200 kmaster kmaster
  172.31.40.40 kworker1 kworker1
  172.31.40.240 kworker2 kworker2

 #yum install -y -q yum-utils device-mapper-persistent-data lvm2
 #yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 #yum install -y -q docker-ce
  yum install docker -y
  systemctl start docker
  systemctl enable docke
  setenforce 0
  sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux

  systemctl disable firewalld
  systemctl stop firewalld

  cat >>/etc/sysctl.d/kubernetes.conf<<EOF
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF

  sysctl --system

  sed -i '/swap/d' /etc/fstab
  swapoff -a

  cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
   [kubernetes]
   name=Kubernetes
   baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
   EOF

yum install -y -q kubeadm-1.18.9 kubelet-1.18.9 kubectl-1.18.9

systemctl enable kubelet
systemctl start kubelet
_________________
#From here following commands should executes on master only
  sudo kubeadm init  <copy o/p as below>

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
-------------------------------------------------------
Then you can join any number of worker nodes by running the following on each as root:  <worker>

kubeadm join 172.31.6.69:6443 --token 3x30l2.t01ru40f8e1rzcie --discovery-token-ca-cert-hash sha256:e1621e5ba3c53566b5908b55a01701b57d39f1abfe7a2c0f40e092b4ac1836ee
  
#copy the kubeadm-token paste it in node servers
  
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml --> MAster

-----------------------
    Sonarqube 
-----------------------
#Here am launching the sonarqube as a docker container
 Launch the t3a.medium instance
  Install JAVA:- yum install java-11* -y
	Install DOCKER:-
	yum install docker -y
	usermod -aG docker ec2-user
	docker run -d -p 9000:9000 --name sonar sonarqube
  
* create a project
* Generate token
* Select maven because we are using java project...based on the language we have to choose
   mvn sonar:sonar/
     -Dsonar.projectKey=
     -Dsonar.host.url=
      -Dsonar.login=
---------------------
       nexus
----------------------
 # we can create a seperate ec2-server or else in sonarqube instance also we will install
  Install JAVA:- yum install java-11* -y
	Install DOCKER:-
	yum install docker -y
  docker run -d -p 8081:8081 -p 8082:8082 -p 8085:8085 --name nexus sonatype/nexus:oss
  
  http://YOUR-SERVER-PUBLIC-IP:8081/nexus
  
  #configuring
   =>Create a private hosted repo for snapshot
   =>Create a private hosted repo for release
   =>Create a proxy repo accessing central repo
   =>Create a group repository assign the above 3 as a group
  
  
  ##After installing and configuring evrything go to the jenkins dashboard
  * In your Jenkins console click on New Item from where you will create your first job
  * After you click on New Item , You need to choose an option freestyle project with name and save
  * In the configuration section select SCM and you will add the git repo link and save it.
  * Then you will select Build option and choose to Execute shell
  * Provide the shell commands. 
    shell command
   mvn clean install
docker build -t java-web-app-cicd:latest .
if (docker ps -a | grep 'java-web-app-cicd')
then
  docker stop java-web-app-cicd
  docker rm -f java-web-app-cicd
fi
docker run -d -p 8888:8080 --name java-web-app-cicd java-web-app-cicd
mvn test
docker login -u <your docker user name> -p <your password>
docker commit java-web-app-cicd <your docker user name>/java-web-app-cicd:latest
docker push <your docker user name>/java-web-app-cicd:latest

kubectl delete deploy --all
kubectl create -f deploy-tomcat.yaml
  
  
###We can give poll scm for Continuous integration and deployment purpose.
  
  
#################################################################################################################################################################################
Using ansible also we can deploy the our source code into the multiple production servers
  
For that we need an Ansible master and multiple slave servers and Jenkins server
  
first we have to do the SSH for all the servers

to do the CI CD follow the procedure:
  1.SSH
  2.launch jenkins dashboard
  3.configure system -> SSH configuration
  4.manage jenkins --> manage plugins --> publishover ssh install
  5.define the hosts file paste the private ips as a group
  6.write a playbook in ansible master 
      - define hosts
               tasks
               name
                   src
                   dest
  7.create a job in jenkins dashboard
  8.provide github url
  9.pollscm
  10.maven phase
  11.post buils step --> give credentials
  12.apply & save
  13.build the job
  14.so whatever time we are providing in poll scm at that particular interval jenkins will automatically check for changes in the github and if there is any change               automatically it will build the code and deploy it into the production servers.
  
  
 
  
  
