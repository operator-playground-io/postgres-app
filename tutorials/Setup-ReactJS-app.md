
### 1 - Setup the application

Get the application
```execute
cd /home/student/projects && git clone https://github.com/operator-playground-io/crunchy-postgres-sample.git
```

Initialise the Database
```execute
export PGPASSWORD=$(kubectl get secret postgres-config -n pgo -o=jsonpath='{.data.POSTGRES_PASSWORD}' | base64 --decode)
echo $PGPASSWORD
cd /home/student/projects/crunchy-postgres-sample
PGPASSWORD=password psql -U pguser -h $ip_addr -p $port contacts < initialize-db.sql 2>output.txt
```

Update the configuration of the application

```execute
rm -rf /home/student/projects/crunchy-postgres-sample/k8s/contacts-pgcluster.yaml 
rm -rf /home/student/projects/crunchy-postgres-sample/k8s/contacts-service.yaml
cd /home/student/projects/crunchy-postgres-sample/k8s && sed -i "s|contacts-db.pgo.svc.cluster.local|contacts|" contacts-backend.deployment.yaml && sed -i "s|contacts.pgo.svc.cluster.local|contacts|" contacts-backend.deployment.yaml
echo "" >> contacts-frontend.deployment.yaml
echo '        - name: REACT_APP_SERVER_URL' >> contacts-frontend.deployment.yaml 
echo "          value: \"http://${ip_addr}:30456/contacts\"" >> contacts-frontend.deployment.yaml

cd /home/student/projects/crunchy-postgres-sample/frontend && export backend_port=30456 && sed -i "s|ip|$ip_addr|" .env && sed -i "s|port|$backend_port|" .env
skaffold config set default-repo localhost:5000
```

### 2 - Deploy the application

Start the application. This step will take ~5 mins.
```execute
cd /home/student/projects/crunchy-postgres-sample
skaffold run -n pgo
```

### 3 - Verify the setup is complete

Execute the below command. The result should have deployments for Database, backend and frontend running.
```execute
kubectl get deployments -n pgo
```