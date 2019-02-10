# EFS Provisioner example 

1. Create your kubernetes cluster (either using EKS or manually on EC2 instances).

2. Note down the VPC details, zone and security groups 

3. Start creating one EFS FS 

    a. Select the same region as the region of your cluster. 
    
    b. On the Services menu - select EFS
    
    c. Create FilsSystem
    
    d. Select the VPC on which your kubernetes cluster lies 
    
    e. Select the zones on which you want the efs 
    
    f. Add the default security group of the zone + the security group of each EC2 instances that wants to connect to the EFS 
    
    g. Add tags (optional) and review your changes 
    
    h. Create the Filesystem and note down -
    
        1. FileSystem ID 
        
        2. DNS Name 
        
4. Create Mounts on the cluster 

    a. mkdir /mnt on all nodes that need the mount 
    
    b. apt-get install nfs-common on all required nodes 
    
    c. Mount the EFS volume to /mnt directory on the machines - 
    
        sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport EFS_DNS:/ /mnt 
        
    d. Verify by running df /mnt - You should see the EFS Filer associated with /mnt 
    
5. Clone the repo - git clone https://github.com/kubernetes-incubator/external-storage.git

6. cd provisioner/external-storage/aws/efs/deploy

7. Edit the manifest file with the below details 

    a.  vi manifest.yaml
    
    b. file.system.id: EFS ID from AWS 
    
    c. aws.region: Region for EFS which should be the same as region of the cluster 
    
    d. server: DNS for EFS 
    
    e. path: /mnt ( This corresponds to DNS:/mnt which translates to /mnt/mnt on local host as /mnt corresponds to EFS_DNS:/ and EFS_DNSL:/mnt = /mnt/mnt )
    
    f.  storage: 1Mi - can be changed as per need for the claim 
    
    g. Save the file 

8. mkdir /mnt/mnt - This should get reflected on all servers on the cluster which has the mount

9. Give appropriate permissions to /mnt/mnt depending on which user your pod is running on 

10. If your cluster has RBAC enabled or you are running OpenShift you must authorize the provisioner. If you are in a namespace/project other than "default" edit deploy/rbac.yaml

11. If you are using kube-system : edit the clusterrolebinding system:controller:endpoint-controller to add the service account that you want to use 

      - kind: ServiceAccount
      
        name: default
        
        namespace: kube-system
        
12. kubectl create -f manifest.yaml  ( or kubectl create -f manifest.yaml -n=NAMESPACE) to create the provisioner in your default NS or the NS of your choice 

13. Verify the pvc  - kubectl get pvc -n=kube-system

14. You now have the PVC which can be used in any application 

15. ls -ltra /mmt/mnt you should see the efs pvc directory created 

    
