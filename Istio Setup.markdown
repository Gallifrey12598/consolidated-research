### This file contains the steps and workarounds I implimented while installing Istio onto CentOS vm

# Download Istio

	$ curl -L https://istio.io/downloadIstio | sh -
	
## Command above will download the latest release for your OS. You can pass veriables onto the command. For example to download 1.13.3.x86_64
	$ curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.13.3 TARGET_ARCH=x86_64 sh -
	
## Now we need to move to the istio package directory ***SEE NOTE BELOW***

### NOTE. I had to download the package as root. This caused issues further down the line cause user could not access the file. Befor moving on I recommend doing the following.

	$ cp -r istio-1.13.3/ /home/user
	$ cd /home/user
	$ chmod -R 755 istio-1.13.3/
	## This allows you to finish up setup as user and not as root. This I have done because root does not recognize the kubectl command. This is probably just for my system, but this is how I worked around the issue
	
## Move into the istio package directory and we are going to add istioctl to our client path
	$ cd istio-1.13.3
	$ export PATH=$PWD/bin:$PATH
## Now we will be able to use the istioctl command

## Install Istio
	## On install we will be setting the profile demo. This has a good set of defaults for this example. 
		## See Additional Information at EOF for further configuration profiles 
	$ istioctl install --set profile=demo -y
	
### NOTE since we are using K3s and not Kubernetes you will have an issue attempting to run the above command. It will pop with error
		*** Error: Get "http://localhost:8080/api?timeout=32s": dial tcp [::1]:8080: connect: connection refused ***
		##This is because Kubernetes uses port 8080, but K3s uses 6443. We will need to update the KUBECONFIG to specify port 6443
		
	$ export KUBECONFIG=/etc/rancher/k3s/k3s.yaml  #You may need to run this command as root. 
	$ istioctl install --set profile=demo -yaml
# This may take some time to complete. Wait till the installation has completed. Once completed do the following to add a namespace lable
	$ kubectl label namespace default istio-injection=enabled

## If you currently have any pods running on the cluster you may encounter issues while installing. I had to delete all pods, replicasets and deployments to get this to work. 
## Mine were just test deployments so it was no issue to delete them. You may first need to cordon the node, delete the pods, run the install then uncordon the node. I have not tried that so it may or may not work

# We are not going to deploy a sample application.

	$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
	
# The application will start. As each pod becomes ready, the Istio sidecar will be deployed along with it

	$ kubectl get services
	and
	$ kubectl get pods
	
# Rerun the pervious 2 commands and wait untill all pods report Ready 2/2 and status RUNNING before moving onto next steps
	
# Verify everything is working correctly up to this point. Run this command to see if the app is running inside the cluster and serving HTML pages by checking for the page title in the response:

	$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
		<title>Simple Bookstore App</title>
		
# Open the application to outside traffic 
	# Associate the application with Istio Gateway
		$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
			gateway.networking.istio.io/bookinfo-gateway created
			virtualservice.networking.istio.io/bookinfo created
	# Ensure there are no issues with the configuration
		$ istioctl analyze
			✔ No validation issues found when analyzing namespace: default.

# Determine the Ingress IP and Ports
	
	$ kubectl get svc istio-ingressgateway -n istio-system

		NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
        istio-ingressgateway   LoadBalancer   172.21.109.129   130.211.10.121  80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
		
# If the EXTERNAL-IP value is set, your environment has an external load balancer that you can use for the ingress gateway. 

# If the EXTERNAL-IP value is <none> (or perpetually <pending>), your environment does not provide an external load balancer for the ingress gateway. In this case, you can access the gateway using the service’s node port.

# Follow these instructions if you have determined that your environment has an external load balancer

#  Follow these instructions if your environment does not have an external load balancer and choose a node port instead.

	# Set the Ingress ports
	
	$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
	
    $ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
	
	$ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')

# See additional information at EOF for additional enviroment setups for Ingress ports.
	
	# Set Gateway URL
		
	$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

	# Ensure an IP address and port were sucessfully assigned
	
	$ echo "$GATEWAY_URL"

	# Run the following command ro retrieve the external ipaddress of the bookstor application
	
	$ echo "http://$GATEWAY_URL/productpage"

	# Paste the output of the previous command into your web browser and confirm Bookinfor product page is displayed. 
	
	
	
*** ADDITIONAL INFORMATION ***
Additional profile configuration URL: https://istio.io/latest/docs/setup/additional-setup/config-profiles/

Additional enviroment setups
	For use for Set Ingress Port section only. 
	
	First two commands are the same for every enviroment 
	
	Google Kubernetes Enviroment (GKE)
		$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
        $ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
		$ export INGRESS_HOST=workerNodeAddress
	
		Further firewall rules need to be implimented
		$ gcloud compute firewall-rules create allow-gateway-http --allow "tcp:$INGRESS_PORT"
        $ gcloud compute firewall-rules create allow-gateway-https --allow "tcp:$SECURE_INGRESS_PORT"
	
	IBM Cloud Kubernetes Enviroment
		$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
		$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
		$ ibmcloud ks workers --cluster cluster-name-or-id
        $ export INGRESS_HOST=public-IP-of-one-of-the-worker-nodes
	
	Docker for Desktop
		$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
		$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
		$ export INGRESS_HOST=127.0.0.1


	
	

	
	