# Jira on GKE

These are my notes on setting up a Jira Server on Google Cloud Platform and k8s.

## Overview

- This guide is only showing the high level steps to provision a Jira setup for GKE
- This is production ready. I used Jira like this in production.

## What makes this work

- Deployment for the Jira server in the latest version. To customize this, edit the DOCKERFILE in this project
	* [Atlassian JIRA Software in a Docker container](https://hub.docker.com/r/cptactionhank/atlassian-jira-software)
- Persistent disk (PVC/PV) for `jira-home` directory
 	* [NFS-Server-Provisioner](https://gitlab.com/bruhsb/k8s-codes/tree/master/nfs-server-provisioner)
- CloudSQL Proxy like a container sidecar
	* [CloudSQL Proxy Docs.](https://cloud.google.com/sql/docs/postgres/sql-proxy)
- Nginx for proxy like a container sidecar

## Usage

There are data to be "customized" manually:

- **PROJECT_NAME:REGION:INSTANCE_NAME** - jira-deployment.yaml
- **JIRA_URL** - jira-deployment.yaml and jira-configmap-server.yaml

**NOTE:** I'm going to start from the premise that there is a Postgresql database instance already configured.

### Jira Deployment

Run in project folder:

```
kubectl create ns jira-cloud && kubectl -n jira-cloud create -f ./
```

This may take a while! Currently this container is big! Like `873 MB` big. I'll improve that later.

```
kubectl get pods
```
```
NAME                          READY   STATUS              RESTARTS   AGE
jira-cloud-7f8dcd97b9-lmhg6   0/3     ContainerCreating   0          2s
```

Once the download completes you'll be good to go:

```
$ kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
jira-cloud-7f8dcd97b9-lmhg6   3/3     Running   0          23h
```

### Configure Jira

Access your Jira install using a local port forward. You don't want to get hacked right out of the gate.

```
kubectl port-forward jira-cloud-7f8dcd97b9-lmhg6 8080:80
```
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Visit http://127.0.0.1:80 in your browser and complete the initial setup.

![Jira Setup](images/jira-1.png)
![Jira Setup](images/jira-2.png)
![Jira Setup](images/jira-3.png)
![Jira Setup](images/jira-4.png)

### External Service

Once you have Jira all setup you can expose it on the public internet. This setup is secure so you'll need to do some extra work get HTTPS working. Just create a secret with the name "tls-https".

```
kubectl get svc -n jira-cloud
```
```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                      AGE
jira-cloud   LoadBalancer   10.59.247.201   xxx.xxx.xxx.xxx   443:31724/TCP,80:30625/TCP   12d

```

At this point you can visit Jira on http://xxx.xxx.xxx.xxx or https://xxx.xxx.xxx.xxx .

## References

Project forked of [Jira on kubernetes](https://github.com/kelseyhightower/jira-on-kubernetes)
