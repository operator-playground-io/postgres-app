### Initialize variables

Execute the below commnds to initialize the variables and create the directories. 

**Note:** If you refresh the lab then, initialise the variables and then proceed to Step where you left.

```execute
mkdir -p /home/student/code-server/postgresql-application
cd /home/student/code-server/postgresql-application
export ip_addr=$(ifconfig eth0 | grep inet | awk '{print $2}' | cut -f2 -d:)
echo $ip_addr
```

### 1 - Create the namespace

```execute
kubectl create ns pgo
```

### 2 - Create the secret

Create the secret with Database configuration
```execute
kubectl create secret generic postgres-config --from-literal=POSTGRES_DB=contacts --from-literal=POSTGRES_USER=pguser --from-literal=POSTGRES_PASSWORD=password -n pgo
```

### 3 - Create the Persistent Volumes
Create Persistent Volume and Persistent Volume Claims.

```execute
cat <<EOF>volume.yaml
---
     kind: PersistentVolume
     apiVersion: v1
     metadata:
        name: postgres-pv-volume
        labels:
          type: local
          app: postgres
     spec:
        storageClassName: manual
        capacity:
          storage: 500Mi
        accessModes:
         - ReadWriteMany
        hostPath:
          path: "/mnt/data"
---
     kind: PersistentVolumeClaim
     apiVersion: v1
     metadata:
        name: postgres-pv-claim
        namespace: pgo
        labels:
          app: postgres
     spec:
        storageClassName: manual
        accessModes:
          - ReadWriteMany
        resources:
          requests:
            storage: 100Mi
EOF
```

```execute
kubectl apply -f volume.yaml
```

### 4 - Create the Database

Create the deployment that creates the Database instance.
```execute
cat <<EOF>deployment.yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        labels:
          app: postgres
        name: postgres
        namespace: pgo
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: postgres
        strategy: {}
        template:
          metadata:
            labels:
              app: postgres
          spec:
            containers:
            - image: postgres:10.19
              name: postgres
              resources: {}
              ports:
                  - containerPort: 5432
              envFrom:
                  - secretRef:
                      name: postgres-config
              volumeMounts:
                  - mountPath: /var/lib/postgresql/data
                    name: postgredb
            volumes:
              - name: postgredb
                persistentVolumeClaim:
                  claimName: postgres-pv-claim
EOF
```

```execute
kubectl apply -f deployment.yaml
```

### 5 - Create the Database conectivity

Create the service to communicate with the Database.
```execute
cat <<EOF>service.yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: contacts
        namespace: pgo
        labels:
          app: postgres
      spec:
        type: NodePort
        ports:
         - port: 5432
        selector:
          app: postgres
EOF
```

```execute
kubectl apply -f service.yaml
```
