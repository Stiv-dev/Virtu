# Screenshots

To help review your deployment, please include the following screenshots in this directory::

## Deployment Pipeline

- DockerHub showing containers that you have pushed
- Docker containers created by docker compose up -d command (docker ps)

## Kubernetes

- To verify Kubernetes pods are deployed properly

```bash
kubectl get pods
```

- To verify Kubernetes services are properly set up

```bash
kubectl describe services
```

- To verify that you have set up logging with a backend application

```bash
kubectl logs {pod_name}
```

## Kubernetes Nodes

- Screenshot of your master and worker nodes
