#update yum componentes

	$yum check-update
	
#install Dependencies required for Docker

	$yum install -y yum-utils device-mapper-persistent-data lvm2
	
#Add Docker Repository

	$yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	
#Install Docker using Yum. For this I just used docker and not docker-ce docker-ce-cli containerd.io or docker-compose-plugin.

# I have not attempted it with thoes so attempt at your own risk. Commands are *_*  

	$yum install docker -y       ---> This is the command I used

	*$yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin*
	
#Manage Docker Service

	$systemctl start docker
	$systemctl enable docker
	$systemctl status docker
	
#After docker has been installed we can move onto downloading and installing k3s

#For the installation to work properly I had to disable the firewall 
		
	$systemctl disable firewalld --now
	
		#If you don't want to disable the firewall you can do the following commands. For simplicity I just disabled the firewall
			$systemctl disabled nm-cloud-setup,service nm-cloud-setup.timer
			$reboot
	
#Once the firewall was disabled I rand the following command to install K3s

	$curl -sfL https://get.k3s.io | sh -        ***SEE BELOW NOTE FOR ADDITIONAL INSTALL INFORMATION***

#Once finished installing, I confirmed it installed corectly with the following command

	$kubectl get node     ***SEE NOTE BELOW***
	
	#I was having issues running kubectl. I kept getting
		"Unable to read /etc/rancher/k3s/k3s.yaml" "open /etc/rancher/k3s/k3s.yaml: Premission denied
	#There are several ways to fix this issue. The one I went with was
		$sudo chmod 644 /etc/rancher/k3s/k3s.yaml
	#I have also found you can run the install script with this veriabl already in place
		$curl -sfL https://get.k3s.io | sh -s --write-kubeconfig-mode 644 
		or
		$curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -s -
	#I have not verified the two above commands cause I had k3s already installed so use at own risk. 
	
	
#I saw the following confirming proper installation

	NAME                       STATUS      ROLES                    AGE       VERSION
	localhost.localdomain      Ready       control-plane, master    [time]    v1.23.6+k3s1
	
#I applied a test deployment example to make sure things are working as intended

	$kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
	
	*feedback* deployment.apps/nginx-deployment created

#The following commands and feedback allowed me to verify the deployment of files

	$kubectl get deployments
	*feedback* 
		NAME                READY  UP-TO-DATE  AVAILABLE     AGE
		nginx-deployment    0/3    0           0             [time]
		
	$kubectl get pods
	*feedback*
		NAME                                 READY    STATUS    RESTARTS      AGE
		nginx-deployment-9456bbbf9-4vwqg     1/1      Running   0             [time]
		nginx-deployment-9456bbbf9-mrq9j     1/1      Running   0             [time]
		nginx-deployment-9456bbbf9-9ppqk     1/1      Running   0             [time]
#I continued these commands untill i got the "kubectl get deployments" command to show 3/3 ready, 3 up-to-date and 3 available

##At this point I know I got k3s installed and working as desired. 

