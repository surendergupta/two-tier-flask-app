 
# Flask App with MySQL Docker Setup

This is a simple Flask app that interacts with a MySQL database. The app allows users to submit messages, which are then stored in the database and displayed on the frontend.

## Prerequisites

Before you begin, make sure you have the following installed:

- Docker
- Git (optional, for cloning the repository)

## Setup

1. Clone this repository (if you haven't already):

   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   ```

2. Navigate to the project directory:

   ```bash
   cd your-repo-name
   ```

3. Create a `.env` file in the project directory to store your MySQL environment variables:

   ```bash
   touch .env
   ```

4. Open the `.env` file and add your MySQL configuration:

   ```
   MYSQL_HOST=mysql
   MYSQL_USER=your_username
   MYSQL_PASSWORD=your_password
   MYSQL_DB=your_database
   ```

## Usage

1. Start the containers using Docker Compose:

   ```bash
   docker-compose up --build
   ```

2. Access the Flask app in your web browser:

   - Frontend: http://localhost
   - Backend: http://localhost:5000

3. Create the `messages` table in your MySQL database:

   - Use a MySQL client or tool (e.g., phpMyAdmin) to execute the following SQL commands:
   
     ```sql
     CREATE TABLE messages (
         id INT AUTO_INCREMENT PRIMARY KEY,
         message TEXT
     );
     ```

4. Interact with the app:

   - Visit http://localhost to see the frontend. You can submit new messages using the form.
   - Visit http://localhost:5000/insert_sql to insert a message directly into the `messages` table via an SQL query.

## Cleaning Up

To stop and remove the Docker containers, press `Ctrl+C` in the terminal where the containers are running, or use the following command:

```bash
docker-compose down
```

## To run this two-tier application using  without docker-compose

- First create a docker image from Dockerfile
```bash
docker build -t flaskapp .
```

- Now, make sure that you have created a network using following command
```bash
docker network create twotier
```

- Attach both the containers in the same network, so that they can communicate with each other

i) MySQL container 
```bash
docker run -d \
    --name mysql \
    -v mysql-data:/var/lib/mysql \
    --network=twotier \
    -e MYSQL_DATABASE=mydb \
    -e MYSQL_USER=root \
    -e MYSQL_ROOT_PASSWORD=admin \
    -p 3306:3306 \
    mysql:5.7

```
ii) Backend container
```bash
docker run -d \
    --name flaskapp \
    --network=twotier \
    -e MYSQL_HOST=mysql \
    -e MYSQL_USER=root \
    -e MYSQL_PASSWORD=admin \
    -e MYSQL_DB=mydb \
    -p 5000:5000 \
    flaskapp:latest

```

## Notes

- Make sure to replace placeholders (e.g., `your_username`, `your_password`, `your_database`) with your actual MySQL configuration.

- This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.

- Be cautious when executing SQL queries directly. Validate and sanitize user inputs to prevent vulnerabilities like SQL injection.

- If you encounter issues, check Docker logs and error messages for troubleshooting.

## kubeadm Installation on ubuntu v1.29.3

1. Install k8s for master and woorker node (1.29)
   
 ```
	1.1 sudo apt-get update
	1.2 sudo su
		swapoff -a; sed -i '/swap/d' /etc/fstab
		exit
	1.3 cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
		overlay
		br_netfilter
		EOF
	1.4 sudo modprobe overlay
	1.5 sudo modprobe br_netfilter
	1.6 cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
		net.bridge.bridge-nf-call-iptables  = 1
		net.bridge.bridge-nf-call-ip6tables = 1
		net.ipv4.ip_forward                 = 1
		EOF
	1.7 sudo sysctl --system
	1.8 sudo apt update
	1.9 sudo apt-get install -y apt-transport-https ca-certificates curl gpg
	1.10 curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	1.11 echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
	1.12 sudo apt-get update
	1.13 sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
	1.14 sudo apt install docker.io -y
	1.15 sudo mkdir /etc/containerd
	1.16 sudo sh -c "containerd config default > /etc/containerd/config.toml"
	1.17 sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
	1.18 sudo systemctl restart containerd.service
	1.19 sudo systemctl restart kubelet.service
	1.20 sudo systemctl enable kubelet.service
	1.21 sudo systemctl status kubelet

```

2. Install kubeadm for master node (1.29)
   
```

	2.1 sudo kubeadm config images pull
	2.2 sudo kubeadm init
	2.3 mkdir -p $HOME/.kube
	2.4 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	2.5 sudo chown $(id -u):$(id -g) $HOME/.kube/config
	2.6 kubectl get po -n kube-system
	2.7 kubectl get --raw='/readyz?verbose'
	2.8 kubectl cluster-info
	2.9 kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
	2.10 kubectl get nodes
	2.11 kubectl get pods

```

3. Connect worker nodoe to master node (1.29)
   
```
	sudo kubeadm join 172.31.0.150:6443 --token b7lkg1.zple77u200x54sp4 \
        --discovery-token-ca-cert-hash sha256:322a2d650ee08bcd13405663b433851221d23aca9a50cb9cf1aa8f89ed0e8899
```

5. Kubeadm installation done

## Flask App deploy on Master node of kubeadm (1.29)

1. git clone https://github.com/surendergupta/two-tier-flask-app.git

2. cd two-tier-flask-app

3. ls

4. mkdir k8s

5. cd k8s

6. vim two-tier-app-pod.yml

7. cat two-tier-app-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: two-tier-app-pod
spec:
  containers:
	- name: two-tier-app-container
	  image: surendergupta/flaskapp:latest
	  env:
		- name: MYSQL_HOST
		  value: "mysql"
		- name: MYSQL_USER
		  value: "root"
		- name : MYSQL_PASSWORD
		  value: "admin"
		- name: MYSQL_DB
		  value: "mydb"
	  ports:
		- containerPort: 5000
	  imagePullPolicy: Always

```

8. kubectl apply -f two-tier-app-pod.yml

9. vim two-tier-app-deployment.yml

10. cat two-tier-app-deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: two-tier-app-deployment
  labels:
	app: two-tier-app
spec:
  replicas: 1
  selector:
	matchLabels:
	  app: two-tier-app
  template:
	metadata:
	  labels:
		app: two-tier-app
	spec:
	  containers:
		- name: two-tier-app
		  image: surendergupta/flaskapp:latest
		  env:
			- name: MYSQL_HOST
			  value: "myhost"
			- name: MYSQL_PASSWORD
			  valueFrom:
				secretKeyRef:
				  name: mysql-secret
				  key: password
			- name: MYSQL_USER
			  value: "root"
			- name: MYSQL_DB
			  value: "mydb"
		  ports:
			- containerPort: 5000
		  imagePullPolicy: Always

```

11. kubectl apply -f two-tier-app-deployment.yml

12. vim two-tier-app-svc.yml

13. cat two-tier-app-svc.yml
```
apiVersion: v1
kind: Service
metadata:
  name: two-tier-app-service
spec:
  selector:
	app: two-tier-app
  ports:
	- protocol: TCP
	  port: 80
	  targetPort: 5000
	  nodePort: 30004
  type: NodePort

```
14. kubectl apply -f two-tier-app-svc.yml

15. vim mysql-pv.yml

16. cat mysql-pv.yml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
	storage: 256Mi
  volumeMode: Filesystem
  accessModes:
	- ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
	path: /home/ubuntu/two-tier-flask-app/mysqldata

```

17. mkdir -p /home/ubuntu/two-tier-flask-app/mysqldata

18. kubectl apply -f mysql-pv.yml	

19. vim mysql-pvc.yml

20. cat mysql-pvc.yml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
	- ReadWriteOnce
  resources:
	requests:
	  storage: 256Mi

```

21. kubectl apply -f mysql-pvc.yml

22. vim mysql-secret.yml

23. cat mysql-secret.yml
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: kubernetes.io/basic-auth
stringData:
  password: mysecretpassword

```

24. kubectl apply -f mysql-secret.yml

26. vim mysql-deployment.yml

27. cat mysql-deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
	app: mysql
spec:
  replicas: 1
  selector:
	matchLabels:
	  app: mysql
  template:
	metadata:
	  labels:
		app: mysql
	spec:
	  containers:
		- name: mysql
		  image: mysql:latest
		  env:
			- name: MYSQL_ROOT_PASSWORD
			  valueFrom:
				secretKeyRef:
				  name: mysql-secret
				  key: password
		  ports:
			- containerPort: 3306
		  volumeMounts:
			- name: mysqldata
			  mountPath: /var/lib/mysql
	  volumes:
		- name: mysqldata
		  persistentVolumeClaim:
			claimName: mysql-pvc

```

28. kubectl apply -f mysql-deployment.yml

29. vim mysql-svc.yml

30. cat mysql-svc.yml
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
	app: mysql
  ports:
	- port: 3306
	  targetPort: 3306

```

31. kubectl apply -f mysql-svc.yml

32. kubectl get svc

33. copy mysql cluster-IP (10.99.2.221)
    
34. Update mysql-deployment.yml env variable MYSQL_HOST  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: two-tier-app-deployment
  labels:
	app: two-tier-app
spec:
  replicas: 1
  selector:
	matchLabels:
	  app: two-tier-app
  template:
	metadata:
	  labels:
		app: two-tier-app
	spec:
	  containers:
		- name: two-tier-app
		  image: surendergupta/flaskapp:latest
		  env:
			- name: MYSQL_HOST
			  value: "10.99.2.221"
			- name: MYSQL_PASSWORD
			  valueFrom:
				secretKeyRef:
				  name: mysql-secret
				  key: password
			- name: MYSQL_USER
			  value: "root"
			- name: MYSQL_DB
			  value: "mydb"
		  ports:
			- containerPort: 5000
		  imagePullPolicy: Always

```

35. kubectl apply -f mysql-deployment.yml

## Access Your MySQL Instance

1. List the pods:
```kubectl get po```

2. Find the MySQL pod and copy its name by selecting it and pressing Ctrl+Shift+C:

3. Get a shell for the pod by executing the following command:
```kubectl exec --stdin --tty mysql-95499797-49mbz -- /bin/bash```
The pod shell replaces the main shell:
	
4. Type the following command to access the MySQL shell:
```mysql -p```

5. When prompted, enter the password you defined in the Kubernetes secret.

6. create DATABASE mydb;

7. show databases;

8. use mydb

9. Create table as my requirement
    ```
    CREATE TABLE messages(
			id INT AUTO_INCREMENT PRIMARY KEY,
			message TEXT
		);

    ```

## [troubleshoot]

```
kubeadm token create --print-join-command
kubectl get pods  -o wide
sudo ctr -n=k8s.io images list -q
kubectl scale deployment two-tier-app-deployment.yml --replicas=4
kubectl logs mysql-6959fff665-kv5zr
kubectl get events
kubectl get nodes
kubectl get po/pods
kubectl get svc
kubectl cluster-info

```

## [References]

1. kubeadm installation :
```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```

2. mysql in kubernetes deployment :
```
https://medium.com/@midejoseph24/deploying-mysql-on-kubernetes-16758a42a746

```

## [Frontend screenshot]

![image](https://github.com/surendergupta/two-tier-flask-app/assets/20636844/704d54c9-2326-417e-b660-fe21268ec774)


