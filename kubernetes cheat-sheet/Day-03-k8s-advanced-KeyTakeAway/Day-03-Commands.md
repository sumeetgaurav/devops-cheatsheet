# Day 03 Commands

## Navigation and file commands

| Command | Usage | Description |
|---|---|---|
| `ls` | `ls` | List files in the current folder |
| `cd` | `cd devboard`, `cd ..` | Move into a folder, or up one level |
| `cat` | `cat 08-postgres.yml` | Print a file's contents |
| `nano` / `vim` | `nano 10-postgres-pv.yml` | Edit or create a file in a terminal editor |
| `mkdir` | `mkdir k8s` | Create a folder |
| `cp -r` | `cp -r ../devboard/*.yml k8s/` | Copy all YAML manifests into the k8s folder |
| `history` | `history` | Show previously run commands |

## kubectl commands

| Command | Usage | Description |
|---|---|---|
| `kubectl get pv` | `kubectl get pv` | List PersistentVolumes (cluster-wide) |
| `kubectl apply -f` | `kubectl apply -f 08-postgres.yml` | Create or update resources from a manifest |
| `kubectl delete -f` | `kubectl delete -f 05-backend-deployment.yml` | Delete the resources defined in a manifest |
| `kubectl get pods -n` | `kubectl get pods -n devboard-ns` | List pods in the devboard-ns namespace |
| `kubectl get all -n` | `kubectl get all -n devboard-ns` | List pods, services, deployments and so on in the namespace |
| `kubectl logs` | `kubectl logs pod/postgres-0 -n devboard-ns` | View a pod's logs (use it to debug crashing pods) |
| `kubectl exec -it` | `kubectl exec -it pod/postgres-0 -n devboard-ns -- /bin/bash` | Open an interactive shell inside a pod |
| `kubectl port-forward` | `kubectl port-forward svc/frontend-service -n devboard-ns 8081:80 --address 0.0.0.0 &` | Expose the service on local port 8081, on all interfaces, in the background |
