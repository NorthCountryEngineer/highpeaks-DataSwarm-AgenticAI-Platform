# highpeaks-DataSwarm-AgenticAI-Platform

The **highpeaks-DataSwarm-AgenticAI-Platform** deploys the Flowise AI application, which provides a visual interface to create and manage AI workflow **agents**. Flowise is an open-source, low-code tool for building LLM (Large Language Model) flows and agents. In the High Peaks platform, this service allows developers and operators to design AI-driven automation flows (for example, an AI GitOps assistant's logic) through a web interface.

## Repository Structure

```text
highpeaks-DataSwarm-AgenticAI-Platform/
├── README.md           # Overview of the Flowise service deployment
├── k8s/
│   ├── deployment.yaml # Kubernetes Deployment to run the Flowise server
│   └── service.yaml    # Kubernetes Service (NodePort) to expose the Flowise UI
└── .github/
    └── workflows/
        └── ci.yml     # GitHub Actions workflow for validating manifests
```

*(Note: This repository primarily contains configuration files. We are leveraging the official Flowise Docker image, so no application source code is present here.)*

## Deployment and Usage

The provided Kubernetes manifests will deploy Flowise in the cluster:
- The **Deployment** runs the official Flowise Docker image (by default `flowiseai/flowise:latest`). This image contains the Flowise UI and server.
- The **Service** is of type NodePort, exposing Flowise's web interface on a high port (default NodePort **30080**, mapped to Flowise's internal port 3000).

**Running on a local Kind cluster:**
1. Ensure your Kind cluster is running (see the infrastructure repo for setup details). 
2. Load or pull the Flowise image:
   - You can pull the image to your local machine with `docker pull flowiseai/flowise:latest`. 
   - Then load it into Kind: `kind load docker-image flowiseai/flowise:latest`.
3. Apply the manifests to deploy Flowise:
   ```bash
   kubectl apply -f k8s/deployment.yaml
   kubectl apply -f k8s/service.yaml
   ```
4. Once the pod is running, access the Flowise UI. If using the provided Kind configuration, the Flowise UI should be accessible at `http://localhost:8080` (the Kind config maps host port 8080 to the cluster's NodePort 30080). Otherwise, you can use `kubectl port-forward`:
   ```bash
   kubectl port-forward svc/highpeaks-flowise-service 8080:3000
   ```
   Then open `http://localhost:8080` in a browser.

**Persistence:** By default, Flowise may store flows in an internal database or file. In this scaffold, no persistent volume is attached, so any flows you create will not be saved if the pod is restarted. For a production setup, you would attach a PersistentVolume to preserve Flowise's data (or configure an external database if supported).

## CI/CD

Since this repository is primarily configuration:
- The CI workflow (`.github/workflows/ci.yml`) will run linting/validation on the Kubernetes YAML files (ensuring they are syntactically correct).
- In a real scenario, we might also periodically check if the pinned Flowise image tag is updated for security patches, etc., as part of a maintenance pipeline.

Deployment of this service in a GitOps workflow would involve referencing these manifests or Helm chart (if we package one in the future) from the infrastructure repository or ArgoCD. For now, manual deployment (as above) is used.

## Security Considerations

Flowise should ideally be secured in a production environment:
- Access to the Flowise UI might be protected by authentication (e.g., integrated with the Identity service via OIDC or an auth proxy) to prevent unauthorized use.
- Network policies can restrict which services can call the Flowise service, and ingress rules would be locked down if exposed externally.
- Always keep the Flowise image updated to incorporate the latest security fixes from the Flowise project.
