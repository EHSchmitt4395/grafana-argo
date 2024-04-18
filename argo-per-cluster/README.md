# Grafana-Agent-Demo

## Overview
This repository facilitates a fully autonomous deployment of the Grafana agent, complete with metrics scrapers, into a virtual Kubernetes (k8s) environment. This setup is ideal for localized environments like Minikube and k3s. It leverages ArgoCD for orchestrating autonomous Helm deployments and includes tools for encrypting sensitive values.

## Deployment Steps

### Initial Setup
1. **Install Docker Desktop** and enable Kubernetes in the settings.

2. **Generate a GPG Key** for Helm-secrets in a custom ArgoCD implementation. For non-development environments, consider using a cloud-compliant KMS system. ex GCP KMS 

    ```bash
    gpg --full-generate-key --rfc4880   # Do not incorporate a passphrase
    gpg --armor --export-secret-keys <key-id> > key.asc
    ```

3. **Create a Grafana Cloud Account**. Opt for the free tier and follow the "Get Started with Kubernetes" guide. This setup integrates all required scraping utilities for a Kubernetes system. The official Helm chart is available at https://grafana.github.io/helm-charts (k8s-monitoring).

### Encryption of Key Values
1. **Encrypt Your Key Values** in the `deploy/argocd/apps` directory.

2. **Install Sops** for encrypting the API key in the values file.

    ```bash
    wget https://github.com/mozilla/sops/releases/download/v3.8.1/sops-v3.8.1.linux
    chmod +x sops-v3.8.1.linux
    sudo mv sops-v3.8.1.linux /usr/local/bin/sops
    ```

3. **Prepare Your Grafana Values File**. Replace the contents of `grafana-values.enc.yaml` with your API key, cluster name, and host endpoints, which can be found on your Grafana Cloud account or the Grafana Cloud cluster configuration page.

```

cluster:
    name: my-cluster-name
externalServices:
    prometheus:
        host: https://URL
        basicAuth:
            username: "USERNAME"
            password: key=
    loki:
        host: https://URL
        basicAuth:
            username: "USERNAME"
            password: key=
    tempo:
        host: https://URL
        basicAuth:
            username: "USERNAME"
            password: key=
opencost:
    opencost:
        exporter:
            defaultClusterId: my-cluster
        prometheus:
            external:
                url: https://URL
traces:
    enabled: true
grafana-agent:
    agent:
        extraPorts:
            - name: otlp-grpc
              port: 4317
              targetPort: 4317
              protocol: TCP
            - name: otlp-http
              port: 4318
              targetPort: 4318
              protocol: TCP
            - name: zipkin
              port: 9411
              targetPort: 9411
              protocol: TCP

# These mounts are critical for docker desktop and local kubernetes deployments with the docker desktop runtime 
grafana-agent-logs:
    agent:
        mounts:
            dockercontainers: true
            extra:
                - name: dockerlib
                  mountPath: /var/lib/docker
                  # Assuming you only need read access
                  readOnly: true
    controller:
        volumes:
            extra:
                - name: dockerlib
                  hostPath:
                    path: /var/lib/docker
                    type: Directory

```


4. **Encrypt the File** using the GPG key.

    ```bash
    sops --encrypt -i --pgp <key-id> values.enc.yaml
    sops --decrypt -i --pgp <key-id> ./values.enc.yaml
    ```

### Installing Kustomize
1. **Download and Install Kustomize**.

    ```bash
    wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv4.1.2/kustomize_v4.1.2_linux_amd64.tar.gz
    tar xzf kustomize_v4.1.2_linux_amd64.tar.gz
    chmod +x kustomize
    sudo mv kustomize /usr/local/bin/
    ```

2. **Navigate to the Correct Directory for Kustomize Build**.

    ```bash
    cd deploy/argocd
    kubectl create ns argocd
    kubectl -n argocd create secret generic helm-secrets-private-keys --from-file=../../../key/key.asc
kubectl -n argocd create secret generic helm-secrets-private-keys --from-file=../../key.asc       
    kustomize build --load-restrictor LoadRestrictionsNone . | kubectl apply -f - --namespace=argocd
    ```

### Final Steps
1. Retrieve the initial admin password for ArgoCD, set up port forwarding, and apply the remaining ArgoCD resources.

    ```bash
    kubectl get secret -n argocd argocd-initial-admin-secret -o yaml
    echo $password | base64 -d 
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    kustomize build --load-restrictor LoadRestrictionsNone . | kubectl apply -f - --namespace=argocd
    ```

    Note: If your forked repo referencing the Helm values file is private, add credentials in ArgoCD settings repository. See `deploy/argocd/apps/grafana.yaml` for context.

## Important Notes

- **Docker Desktop Kubernetes**: Be aware that many labels used in standard Kubernetes deployments do not exist in Docker Desktop Kubernetes. You may need to adjust dashboard tags accordingly.
- **Underlying CNI Limitations**: The Docker Desktop Kubernetes CNI has limitations, such as less abstracted bandwidth tracking in namespaces.
- **Corrected Recording Rules**: If using Docker Desktop Kubernetes, you might need to modify the recording rules in Prometheus queries. Additionaly you will likely need to edit grafana dashboards to omit the same non existant tags from scraping metrics like cAdvisor 

    ```bash
    sum by (cluster, namespace, pod, container) (
      irate(container_cpu_usage_seconds_total{job!="", image!=""}[5m])
    ) * on (cluster, namespace, pod) group_left(node) topk by (cluster, namespace, pod) (
      1, max by(cluster, namespace, pod, node) (kube_pod_info{node!=""})
    )
    # Remove job!="", image!=""
    ```
etc hosts for nginx loki 
127.0.0.1 loki.example.comk6

which go  install from web not brew 
might need export PATH=$PATH:/usr/local/go/bin
do 
export GOPATH=$HOME/go
export PATH=$PATH:$(go env GOPATH)/bin

cd /Users/ericschmitt/Documents/xk6-loki to run tests 
./k6 run ../grafana-agent-demo/deploy/argocd/k6-test-basic-new-best.js