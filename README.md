# K8S-Demo

A minimal Kubernetes demo: a Node.js web app backed by MongoDB, deployed to a local
[minikube](https://minikube.sigs.k8s.io/) cluster.

## Architecture

```
Browser ──NodePort:30100──▶ webapp-service ──▶ webapp pod (port 3000)
                                                     │
                                                     ▼
                                        mongo-service ──▶ mongo pod (port 27017)
```

| File                 | Resource                          | Purpose                                          |
| -------------------- | --------------------------------- | ------------------------------------------------ |
| `mongo-config.yaml`  | ConfigMap                         | Mongo service URL for the web app                |
| `mongo-secret.yaml`  | Secret                            | Mongo username / password                        |
| `mongo.yaml`         | Deployment + Service (ClusterIP)  | MongoDB, reachable only inside the cluster       |
| `webapp.yaml`        | Deployment + Service (NodePort)   | Web app, exposed on node port `30100`            |

## Prerequisites

- [minikube](https://minikube.sigs.k8s.io/docs/start/)
- `kubectl`

## Deploy

```bash
minikube start
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo.yaml
kubectl apply -f webapp.yaml
```

Verify everything is running:

```bash
kubectl get pods,svc
```

## Accessing the app

The web app is exposed via a NodePort on `30100`.

> **macOS / Windows (docker driver):** The node IP shown by `minikube ip`
> (e.g. `192.168.49.2`) is **not reachable from your browser** — it lives on an
> internal Docker network. Use one of the commands below instead. Directly
> browsing `http://192.168.49.2:30100/` will not work.

```bash
# Opens a tunnel and prints (usually auto-opens) a working URL
minikube service webapp-service
```

or forward a local port:

```bash
kubectl port-forward service/webapp-service 30100:3000
# then open http://localhost:30100/
```

## A note on the Secret

`mongo-secret.yaml` contains **throwaway demo credentials only**
(`mongouser` / `mongopassword`, base64-encoded under `data:`). They are committed
here purely so the demo works out of the box.

**Do not commit real secrets to Git.** For a real project, keep secret manifests
out of version control (e.g. add them to `.gitignore` and commit an
`*.example.yaml` with placeholder values instead), or use a secrets manager.

## Tear down

```bash
kubectl delete -f webapp.yaml -f mongo.yaml -f mongo-secret.yaml -f mongo-config.yaml
```
