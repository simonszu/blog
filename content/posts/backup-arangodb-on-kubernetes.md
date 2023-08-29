---
title: "Backup Arangodb on Kubernetes"
date: 2022-07-26T13:53:58+02:00
draft: false
---

In a recent project, our architect decided to use [ArangoDB](https://www.arangodb.com/) as a database. I am not familiar with that, but it seems like one of these fancy hipster NoSQL stuff - at least it is written in node.js. But who am i to judge?

As the responsible DevOps engineer i needed a solution for backuping the database. There is this fancy [Backup Operator](https://www.arangodb.com/docs/stable/deployment-kubernetes-backup-resource.html) but unfortunately it is not available for the community version of ArangoDB. Of course, my customer, being a huge IT corporation with thousands of employees, did not want to pay money for the main data holding component of this very project. A very reasonable decision. So i needed another solution to backup this.

Fortunately, the ArangoDB container image comes with a handy little binary called `arangodump`` which, surprise surprise, dumps the content of a database to the file system. From there on, we just need to grab it, and push it to the desired destination - in our case, a S3 instance, or Object Block Storage, as the hyperscaling provider of our choice calls it.

To put everything together, i created a kubernetes cron job. It uses a dedicated config map and a dedicated secret. This is the config map:

{{< highlight yaml >}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: backup-config
  namespace: arangodb
data:
  admin-user: root
  db-url: ${endpoint}
  obs-endpoint: ${obs-endpoint}
  obs-bucket: ${obs-bucket}
{{< /highlight >}}

Fill in the variables as designed. As you can maybe see, we are using Terraform to provision all the stuff. `db-url` is the service URL of the ArangoDB deployment, both `obs` variables are for the desired backup destination.

This is the associated secret:

{{< highlight yaml >}}
apiVersion: v1
kind: Secret
metadata:
  name: backup-secret
  type: Opaque
  namespace: arangodb
data:
  access-key: ${accesskey}
  secret: ${secret}
  root-password: ${root-password}
{{< /highlight >}}

In this case, `access-key` and `secret` are the API token of a technical user of the hyperscaler which is allowed to access the block storage and bucket defined in the config map. `root-password` is for the database access.

The cronjob itself consists of two containers. One will dump the DB content to a temporary folder (like described), one will pick up the content and push it to the S3 instance with the help of the [MinIO client](https://docs.min.io/docs/minio-client-complete-guide.html). Check it out:

{{< highlight yaml >}}
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: arangodb-backup-cronjob
  namespace: arangodb
spec:
  schedule: "22 3 * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          name: arangodb-backup-job
          namespace: arangodb
        spec:
          initContainers:
            - name: dump-create
              image: "arangodb:latest"
              args:
                - "arangodump"
                - "--server.endpoint=$(ENDPOINT)"
                - "--server.username=$(USERNAME)"
                - "--server.password=$(PASSWORD)"
                - "--all-databases"
                - "--output-directory=/tmp/dump"
                - "--overwrite"
              volumeMounts:
                - name: dump
                  mountPath: /tmp/dump
              env:
              - name: "PASSWORD"
                valueFrom:
                  secretKeyRef:
                    name: backup-secret
                    key: root-password     
              - name: "USERNAME"
                valueFrom:
                  configMapKeyRef:
                    name: backup-config
                    key: "admin-user"              
              - name: "ENDPOINT"
                valueFrom:
                  configMapKeyRef:
                    name: backup-config
                    key: db-url
          restartPolicy: OnFailure
          containers:
            - name: db-dump-upload
              image: "${docker_registry}minio/mc"
              imagePullPolicy: IfNotPresent
              command: ["/bin/sh","-c"]
              args: ["mc alias set obs $OBS_ENDPOINT $ACCESSKEY $SECRETKEY; mc mirror /tmp/dump obs/$OBS_BUCKET/$(date -I)"]
              volumeMounts:
                - name: dump
                  mountPath: /tmp/dump
              env:
              - name: SECRETKEY
                valueFrom:
                  secretKeyRef:
                    name: backup-secret
                    key: secret                   
              - name: ACCESSKEY
                valueFrom:
                  secretKeyRef:
                    name: backup-secret
                    key: access-key
              - name: OBS_ENDPOINT
                valueFrom:
                  configMapKeyRef:
                    name: backup-config
                    key: obs-endpoint
              - name: OBS_BUCKET
                valueFrom:
                  configMapKeyRef:
                    name: backup-config
                    key: obs-bucket
          volumes:
            - name: dump
              emptyDir: {}
{{< /highlight >}}

Go ahead and change your schedule, and of course the S3 retention policies of the bucket to your need - but that's it.