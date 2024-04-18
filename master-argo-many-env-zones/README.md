# Grafana-Agent-Demo

## Overview
matrix gen resources: 
- https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Matrix/
- https://github.com/argoproj/argo-cd/tree/master/applicationset/examples/matrix 


Great resource for argo with GCP fleet and quick labs 

- https://cloud.google.com/blog/products/containers-kubernetes/building-a-fleet-with-argocd-and-gke 
- https://github.com/GoogleCloudPlatform/gke-poc-toolkit-demos/tree/main/gke-fleets-with-argocd 


This directory serves as a example driven baseline for inspiring SCM structure.  Argo is best utilized as an orchestration tool comparing state with git, so a matrix generator utilizing the argo git generator within an applicationset is the best practice. 

While argo-per-cluster direc is best for single cluster deployments, you can extrapolate to utilize this same model for a management cluster having ownership of an app based cluster. I would suggest this for when only one cluster is within requirement. 

It is of the wiser to utilize this direc and applicationset model for deploying all apps within a direc structure when and if more than one cluster is required. 

In the case of grafana consider a management cluster with argocd + metamonitoring utility to manage the clusters with actual production workload. From here you can deploy jobs for k6 + loadtesting. 

Also please see the gcp git example for a basic rollout implementation. this in conjunction with helm provide advanced tooling and testing capabilities with k6 for introducing upgrades from lower to upper environments. 