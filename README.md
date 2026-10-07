# Image Vulnerabilities for Rancher

Rancher UI extension that shows image vulnerability counts on workload lists. It targets Rancher 2.10 or newer.

The extension does not scan images. [Trivy Operator](https://github.com/aquasecurity/trivy-operator) scans the cluster and writes `VulnerabilityReport` resources. This extension reads those reports through the Kubernetes API that Rancher already proxies.

## How it works

Workload tables get a **Vulns** column. That includes Deployments, DaemonSets, StatefulSets, ReplicaSets, Jobs, CronJobs, and Pods. Pods show the scan of the ReplicaSet, StatefulSet, DaemonSet, or Job that owns them.

The column shows critical, high, and medium counts. A scan with none of those findings shows a green **No vulnerabilities** pill. Low and unknown findings are left out. Click a cell to open the CVE list for that image.

Cluster Explorer also has **Container Security → Vulnerabilities**, with totals for the current namespace filter and the same workloads.

A browser refresh loads the latest `VulnerabilityReport` objects. It does not start a new Trivy scan. Trivy Operator updates reports on its own schedule.

## Screenshots

The extension installed from the Rancher Extensions page:

![Image Vulnerabilities 0.1.8 on the Installed extensions tab](docs/images/extensions-installed.png)

The **Vulns** column on a workload list. Clean images show the green pill. Images with findings show critical, high, and medium counts:

![Workload list with the Vulns column](docs/images/workload-column.png)

**Container Security → Vulnerabilities** in Cluster Explorer:

![Vulnerabilities page with cluster totals](docs/images/vulnerabilities-page.png)

## Install Trivy Operator

Trivy Operator is required. Without it, the column stays empty because there are no `VulnerabilityReport` resources to read.

```bash
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --version 0.37.0
```

Point `kubectl` and Helm at the cluster you want scanned. The chart scans workloads in all namespaces by default.

Check whether an operator is already installed before running that command:

```bash
kubectl get deploy -A | grep trivy
```

## Install the extension

1. In Rancher, open **Extensions**.
2. Open the menu at the top right and choose **Manage Repositories**.
3. Add a repository with this Helm index URL:

   ```text
   https://thiagoloureiro.github.io/rancher-security-chart/
   ```

4. Open the **Available** tab and install **Image Vulnerabilities**.
5. Reload the browser.

The **Vulns** column appears on workload lists, and **Container Security → Vulnerabilities** appears in Cluster Explorer.

## Develop

Use Node.js 24.

```bash
nvm use
yarn install
API=https://<your-rancher> yarn dev
```

Open https://127.0.0.1:8005 and sign in. The extension loads into the development UI automatically.

```bash
yarn test
```
