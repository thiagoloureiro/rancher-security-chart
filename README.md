# Image Vulnerabilities for Rancher

Rancher UI extension that shows Trivy Operator image findings on workloads.

It does not scan images. Trivy Operator writes `VulnerabilityReport` resources, and this extension reads them through the Kubernetes API that Rancher already proxies.

On Deployments, DaemonSets, StatefulSets, ReplicaSets, Jobs, CronJobs, and Pods, the workload table gets a **Vulnerabilities** column:

`C 0  H 2  M 8  L 15`

Click a cell to open the finding list. Cluster Explorer also gets **Security → Vulnerabilities**, with cluster totals and the same detail panel.

This targets Rancher 2.10 or newer (`@rancher/shell` 3.x). Use Node.js 24.

## What you need in the cluster

Install [Trivy Operator](https://aquasecurity.github.io/trivy-operator/) so it creates `aquasecurity.github.io/v1alpha1` `VulnerabilityReport` objects. The extension matches a workload to those reports by namespace, owner, and container image. Deployments are matched to the current ReplicaSet scan. An older ReplicaSet for the same Deployment is ignored when a newer scan exists.

## Develop

```bash
nvm use
yarn install
API=https://<your-rancher> yarn dev
```

Open https://127.0.0.1:8005 and sign in. The extension loads into the development UI automatically.

To exercise only the image-matching logic:

```bash
yarn test
```

## Publish the same way as AlertHawk.Chart

AlertHawk.Chart is a Helm repository on GitHub Pages. This extension is published the same way, except Rancher's workflow builds the JavaScript bundle and the extension chart for you.

1. Push this repository to GitHub.
2. Create an empty `gh-pages` branch. The release workflow refuses to run until that branch exists.

   ```bash
   git checkout --orphan gh-pages
   git rm -rf .
   git commit --allow-empty -m "Initialize GitHub Pages"
   git push -u origin gh-pages
   git checkout main
   ```

3. In the repository settings, open **Pages** and set the source to **Deploy from a branch**, branch `gh-pages`, folder `/ (root)`.
4. Publish a GitHub Release whose tag is the extension folder plus the version in `pkg/rancher-vulnerability-ui/package.json`.

   For the current version that tag is:

   ```text
   rancher-vulnerability-ui-0.1.0
   ```

   `.github/workflows/build-extension-charts.yml` runs on that release and pushes the Helm repository to `gh-pages`.

5. The Pages URL is the repository URL, for example:

   ```text
   https://thiagoloureiro.github.io/rancher-security/
   ```

## Install in Rancher

1. In the local cluster, go to **Apps → Repositories** and add the GitHub Pages Helm URL, the same way you add `https://thiagoloureiro.github.io/AlertHawk.Chart/`.
2. Open **Extensions → Available** and install **Image Vulnerabilities**.
3. Open a downstream cluster. Workload lists show the new column, and **Security → Vulnerabilities** is in Cluster Explorer.

For a local check without a release, build and serve the package:

```bash
yarn build-pkg rancher-vulnerability-ui
yarn serve-pkgs
```

In Rancher, enable **Extension developer features** under your user preferences, then use **Extensions → Developer load** with the URL printed by `serve-pkgs`.
