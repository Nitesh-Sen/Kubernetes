# EKS
- Create cluster
```
eksctl create cluster --name cluster3 --nodes 1 --node-type t3.medium --region us-west-1 --nodegroup-name nodegroup-0.1
```
- Check config contexts. There are a new one context added. Use that context name <cluster name>.
```
kubectl config get-
```
```
kubectl config use-context <CLUSTER_CONTEXT_NAME>
```
- Check current context
```
kubectl config current-context
```
- Create a yml file of storate class and add this [content](https://github.com/Nitesh-Sen/Kubernetes/blob/80e4ce25bab2c1fc566d327e384bf01fd870930a/Wordpress-on-eks-with-EBS%26CLB/sc.yml).
```
vim sc.yml
```
- Apply sc.yml 
```
kubectl apply -f sc.yml 
```
- create a yml file of persistence volume claim and add this [content](https://github.com/Nitesh-Sen/Kubernetes/blob/80e4ce25bab2c1fc566d327e384bf01fd870930a/Wordpress-on-eks-with-EBS%26CLB/pvc.yml).
```
vim pvc.yml
```
- Apply this pvc.yml
```
kubectl apply -f pvc.yml 
```
- After do this step check your persistent volume. That will show just pending not created. It has to create the volume. But there we have to make the EBS connection with eks privatly and give to IAM role to that cluster.
```
kubectl get pvc
```
- Follow there command to do this and this is also available on this [link](https://stackoverflow.com/questions/75758115/persistentvolumeclaim-is-stuck-waiting-for-a-volume-to-be-created-either-by-ex).
```
eksctl utils associate-iam-oidc-provider --region=<CLUSTER_REGION> --cluster=<CLUSTER_NAME> --approve
```
```
eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster <CLUSTER_NAME> --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve --role-only --role-name AmazonEKS_EBS_CSI_DriverRole
```
```
eksctl create addon --name aws-ebs-csi-driver --cluster <CLUSTER_NAME> --service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/AmazonEKS_EBS_CSI_DriverRole --force
```
- After successfull this command check you persistent volume will be created.
```
kubectl get pvc
```
```
kubectl get pv
```
- Now create mysql.yml file and add this [content](https://github.com/Nitesh-Sen/Kubernetes/blob/80e4ce25bab2c1fc566d327e384bf01fd870930a/Wordpress-on-eks-with-EBS%26CLB/mysql.yml). Then apply this.
```
vim mysql.yml
```
```
kubectl apply -f mysql.yml
```
> check your pod has created successfully. 
After mysql create wordpress.yml file and add this [contenct](https://github.com/Nitesh-Sen/Kubernetes/blob/80e4ce25bab2c1fc566d327e384bf01fd870930a/Wordpress-on-eks-with-EBS%26CLB/wp.yml).
```
vim wp.yml
```
```
kubectl apply -f wp.yml
```
```
kubectl get pods
```
- Now you create service with these both pod. Expose both pod and create services.
```
kubectl expose pod <MYSQL_POD_NAME> --port 3306 --name mysql-svc
```
```
kubectl expose pod <WORDPRESS_POD_NAME> --port 80 --target-port 80 --type=LoadBalancer --name wordpress-svc
```
- Check services
```
kubectl get svc
```
- Now describe your wordpress service. which is the loadbalancer. Take dns name and browse it on browser.
```
kubectl describe svc wordpress-svc
```
> NOTE: If your loadbalancer dns is not working. Remember it this is not any error of security group or any other. Refresh the page many time that will work after some tries. 

- And lastly describe mysql pod for get the ip of mysql.
```
kubectl describe pod mysql | grep "IP:        "
```
> Take this ip and add there in the browser.
>
> *Your website will be work after these steps.*
  
### Thanks For Visit This Page.
