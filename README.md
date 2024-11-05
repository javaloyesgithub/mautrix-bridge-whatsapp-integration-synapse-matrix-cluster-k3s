# mautrix-bridge-whatsapp-integration-synapse-matrix-cluster-k3s
k3s cluster mautrix bridge whatsapp integration synapse matrix 

Mautrix WhatsApp Bridge Integration with Synapse on Kubernetes (k3s)

This tutorial details the integration between the Mautrix WhatsApp bridge and Synapse in a Kubernetes cluster using k3s. This setup allows smooth communication between Matrix and WhatsApp in a self-managed infrastructure.
Swifth to Spanish
Step 1: Namespace and Volume Configuration

Create a namespace to organize the Matrix resources.

kubectl create namespace namespace_name

Set up a persistent volume for Synapse:

apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: synapse-pv-claim
          namespace: namespace_name
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 5Gi
        

Set up a PersistentVolume (PV):

apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: synapse-pv
        spec:
          capacity:
            storage: 5Gi
          accessModes:
            - ReadWriteOnce
          hostPath:
            path: /path/to/synapse/directory
        

Apply the configuration with the command:

kubectl apply -f synapse-pv.yaml

kubectl apply -f synapse-pv-claim.yaml

Step 2: Deploy Synapse with PostgreSQL

Set up the PostgreSQL database:

apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: postgres
          namespace: namespace_name
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: postgres
          template:
            metadata:
              labels:
                app: postgres
            spec:
              containers:
              - name: postgres
                image: postgres:latest
                env:
                  - name: POSTGRES_DB
                    value: "synapse"
                  - name: POSTGRES_USER
                    value: "postgres_user"
                  - name: POSTGRES_PASSWORD
                    value: "postgres_password"
                ports:
                  - containerPort: 5432
                volumeMounts:
                  - name: postgres-data
                    mountPath: /var/lib/postgresql/data
              volumes:
                - name: postgres-data
                  persistentVolumeClaim:
                    claimName: postgres-pv-claim
        

Set up the PersistentVolumeClaim (PVC) for PostgreSQL:

apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: postgres-pv-claim
          namespace: namespace_name
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 5Gi
        

Deploy PostgreSQL:

kubectl apply -f postgres-deployment.yaml

Configure Synapse as a service in the namespace namespace_name.

Synapse Deployment File (synapse-deployment.yaml):

apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: synapse
          namespace: namespace_name
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: synapse
          template:
            metadata:
              labels:
                app: synapse
            spec:
              containers:
              - name: synapse
                image: matrixdotorg/synapse:latest
                ports:
                  - containerPort: 8008
                env:
                  - name: SYNAPSE_SERVER_NAME
                    value: "domain_name"
                  - name: SYNAPSE_REPORT_STATS
                    value: "yes"
                  - name: POSTGRES_DB
                    value: "synapse"
                  - name: POSTGRES_USER
                    value: "postgres_user"
                  - name: POSTGRES_PASSWORD
                    value: "postgres_password"
                  - name: POSTGRES_HOST
                    value: "postgres"
                volumeMounts:
                  - name: synapse-data
                    mountPath: /data
              volumes:
                - name: synapse-data
                  persistentVolumeClaim:
                    claimName: synapse-pv-claim
        

Deploy Synapse:

kubectl apply -f synapse-deployment.yaml

Step 3: Deploy Mautrix WhatsApp

Configure the deployment of Mautrix WhatsApp:

apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: mautrix-whatsapp
          namespace: namespace_name
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: mautrix-whatsapp
          template:
            metadata:
              labels:
                app: mautrix-whatsapp
            spec:
              containers:
              - name: mautrix-whatsapp
                image: dock.mau.dev/mautrix/whatsapp:latest
                env:
                  - name: HOMESERVER_URL
                    value: "http://synapse:8008"
                  - name: DATABASE_TYPE
                    value: "sqlite"
                  - name: DATABASE_PATH
                    value: "/data/database.db"
                volumeMounts:
                  - name: mautrix-data
                    mountPath: /data
              volumes:
                - name: mautrix-data
                  persistentVolumeClaim:
                    claimName: mautrix-pv-claim
        

Set up the PersistentVolumeClaim (PVC) for Mautrix WhatsApp:

apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: mautrix-pv-claim
          namespace: namespace_name
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
        

Deploy Mautrix WhatsApp:

kubectl apply -f mautrix-whatsapp-deployment.yaml

Step 4: Modifying Configuration Files

Make the necessary modifications to the configuration files to ensure that the Mautrix WhatsApp bridge integrates correctly with Synapse and PostgreSQL.

Ensure that the homeserver.yaml file in the Synapse deployment has the necessary configurations for bridge integration, including:

app_service_config:
          - name: "Mautrix WhatsApp"
            type: "mau-trix"
            ...  # Rest of the configuration
        

Remember to restart the pods after making changes to the configuration:

kubectl rollout restart deployment/synapse -n namespace_name

kubectl rollout restart deployment/mautrix-whatsapp -n namespace_name

