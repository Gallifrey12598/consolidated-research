#K3s Helpful Information

	##During my install and setup of a K3s Server I ran into some helpful things during my research
	##I was having an issue where I wanted to delete a pod and deployment for good, but pods kept getting created. 
	
#To delete a pod
	
		$kubectl get pods
	#Verify the pod name you want deleted
		$kubectl delete pod {POD-NAME-HERE}
	#If sucessfully deleted then the command line should echo that
	
	#Pod just keeps getting recreated
	
		$kubectl get deployments
	#Verify the name of the deployment you wish to delete
		$kubectl delete deployment/{DEPLOYMENT-NAME}
	#If sucessfully deleted then the command line should echo that. Repeat steps to delete pod
	
#If the pods are still being created thendo the following

		$kubectl get all
	#This will list out all pods, services, daemonset and replicaset. Look through and find anything to do with the pod you are trying to delete. Once found.
	
		$kubectl get all --all-namespaces ##This will list the name of everything above
		$kubectl get all -n {NAME} #The specific name of the pod/service/daemonst/replicaset you want to delete
		$kubectl delete {daemonset.apps/replicaset.apps/services/deployment.apps/pod}/{NAME-OF-SPECIFIC-SERVICE} 
		
		##If completed sucessfully then the command line should echo that 
		