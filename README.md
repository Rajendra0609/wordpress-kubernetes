1. Start Minikube
Make sure it’s running and has the resources:

bash
Copy
Edit
minikube start --memory=4096 --cpus=2
2. Create Necessary Directories on Minikube Node
Your PVs use hostPath, which must exist on the Minikube VM.

SSH into Minikube and create the required folders:

bash
Copy
Edit
minikube ssh
sudo mkdir -p /mnt/data/wp-pv
sudo mkdir -p /mnt/data/mysql
exit

✅ Troubleshooting Tips
Check pod logs:

bash
Copy
Edit
kubectl logs -n wordpress deploy/mysql
kubectl logs -n wordpress deploy/wordpress
Check MySQL is accepting connections:

bash
Copy
Edit
kubectl exec -n wordpress -it deploy/wordpress -- curl mysql:3306

🔁 Apply the Fix
First, delete the current broken PVCs and PVs:

bash
Copy
Edit
kubectl delete pvc wp-pvc mysql-pvc -n wordpress
kubectl delete pv wp-pv mysql-pv
Apply the fixed PV and PVC YAML:

bash
Copy
Edit
kubectl apply -f fixed-pv-pvc.yaml
Confirm they're bound:

bash
Copy
Edit
kubectl get pvc -n wordpress
kubectl get pv
Expected output:

nginx
Copy
Edit
NAME        STATUS   VOLUME     ...
mysql-pvc   Bound    mysql-pv
wp-pvc      Bound    wp-pv


✅ 1. Install the MySQL client in the WordPress pod (if not already):
If the image doesn't include it (which most WordPress containers don't), here's a workaround:

bash
Copy
Edit
kubectl exec -n wordpress -it deploy/wordpress -- bash
Inside the pod, try installing the MySQL client (Debian-based):

bash
Copy
Edit
apt update && apt install -y default-mysql-client
Then test:

bash
Copy
Edit
mysql -h mysql -u wpuser -pwppass wordpress
This will try to connect to the mysql service in the same namespace, using the credentials you configured in your environment variables.

✅ Success Criteria:
If connection works, you'll see:

sql
Copy
Edit
Welcome to the MySQL monitor...
mysql>
If it fails, you'll get errors like:

Access denied

Can't connect to MySQL server

Unknown host



minikube service wordpress -n wordpress --url

curl http://192.168.49.2:30080
 
curl -I http://192.168.49.2:30080

