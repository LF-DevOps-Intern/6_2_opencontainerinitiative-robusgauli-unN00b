```
< ## K8s Assignment 2 >
 ---------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```

### Assignment 2

1. Deploy Postgres database using PVC & PV cluster

First, we need to create yaml config to create our PV as well as PVC.

```YAML
# /home/tom/assignment/kuber/postgres/post-pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
        name: post-pv
        namespace: db
spec:
        storageClassName: ""
        persistentVolumeReclaimPolicy: Retain
        accessModes:
                - ReadWriteMany
        capacity:
                storage: 256M
        hostPath:
                path: /db

```

```YAML
# /home/tom/assignment/kuber/postgres/post-pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
        name: post-pvc
spec:
        storageClassName: ""
        volumeName: post-pv
        accessModes:
                - ReadWriteMany
        resources:
                requests:
                        storage: 128M
```

![image](https://user-images.githubusercontent.com/23631617/143810592-77b75b08-1d65-46c4-bf31-1341e5b9e1fb.png)

```YAML
# /home/tom/assignment/kuber/postgres/post-cmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
        name: post-cmap
data:
        POSTGRES_PASSWORD: docker
```

```YAML
# /home/tom/assignment/kuber/postgres/post-svc.yaml
apiVersion: v1
kind: Service
metadata:
        name: post-svc
spec:
        type: NodePort
        ports:
                - port: 30432
                  targetPort: 5432
                  nodePort: 30432
```

![image](https://user-images.githubusercontent.com/23631617/143820312-afb70ef0-1e47-4fdb-8408-75c64df60683.png)

```Dockerfile
# /home/tom/Documents/assignment/kuber/postgres/Dockerfile
FROM postgres

ENV POSTGRES_PASSWORD docker
```

```YAML
# /home/tom/Documents/assignment/kuber/postgres/Dockerfile
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depost
spec:
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      volumes:
      - name: post-pv
        persistentVolumeClaim:
                claimName: post-pvc
      containers:
      - name: postgres
        image: kubepost:99
        envFrom:
        - configMapRef:
                name: post-cmap
        volumeMounts:
                - name: post-pv
                  mountPath: /var/lib/postgresql/data
```

##### Deployment
![image](https://user-images.githubusercontent.com/23631617/143823897-6a6e5923-e521-48ce-8b0e-8d26beb4bb45.png)

---

2. Deploy Postgres Client in cluster(psql)

```YAML
# /home/tom/Documents/assignment/kuber/postgres/client/postgres-cl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depostcl
spec:
  selector:
    matchLabels:
      app: postgrescl
  template:
    metadata:
      labels:
        app: postgrescl
    spec:
      containers:
      - name: postgrescl
        image: postgres
        envFrom:
        - configMapRef:
                name: postgrescl-cmap
```

```YAML
# /home/tom/Documents/assignment/kuber/postgres/client/postgrescl-cmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
        name: postgrescl-cmap
data:
        POSTGRES_PASSWORD: docker
```

```bash
kubectl exec -it depostcl-6c66dcb49d-pw9zs -- psql --version
```

##### Deployed psql client
![image](https://user-images.githubusercontent.com/23631617/143827034-cfe1d8ed-7f24-4d11-bc24-4d7a42e6c116.png)

---

3. Connect Postgres database from Postgres Client using core-dns's host name.

##### Connecting using hostname
![image](https://user-images.githubusercontent.com/23631617/143830706-d64e705f-240a-427e-b01c-2eee2bd0cd01.png)

---

4. Create a database(internship) and few tables in database.

```SQL
CREATE database internship;

CREATE TABLE students ( id int, name varchar(40) );
CREATE TABLE mentors ( id int, name varchar(40) );
```
