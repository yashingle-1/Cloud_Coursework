# GenAI Troubleshooting Log

## Overview

Throughout this project, I utilised Microsoft Copilot as an AI-assisted troubleshooting tool to help resolve configuration issues, debug errors, and optimise the deployment pipeline across both Kubernetes environments. The following table documents the key issues encountered, the solutions proposed by the AI, and whether those solutions were effective.

\---

## Troubleshooting Table

|#|Problem Encountered|AI-Proposed Solution|Outcome|
|-|-|-|-|
|1|Helm `repo add` failed with **permission denied** error on the cache folder|Run `sudo chown -R $USER:$USER \~/.cache/helm` and `\~/.config/helm` to fix ownership, then re-add the repository|✅ Worked|
|2|Helm install failed with **Kubernetes cluster unreachable** on K3s environment|Copy the K3s kubeconfig using `sudo cp /etc/rancher/k3s/k3s.yaml \~/.kube/config`, set ownership, and export `KUBECONFIG=\~/.kube/config` in `\~/.bashrc`|✅ Worked|
|3|Minikube failed to start with error **K8S\_APISERVER\_MISSING** — API server process never appeared|Identified root cause as critically low disk space. Proposed freeing space via `docker system prune -a -f`, `apt clean`, and removing unused snap packages|⚠️ Partial — disk cleanup helped but full resolution required adding a second VDI disk|
|4|**Low disk space** on kube-cloud VM due to Ubuntu Desktop ISO being used instead of Server|Suggested removing snap packages (Firefox, Gnome, GTK themes), running Docker and APT cleanup commands, and resizing VDI via VBoxManage|⚠️ Partial — snap removal freed space but VDI resize did not fully apply|
|5|**Second VDI disk** added via VirtualBox to resolve persistent disk space issue|Format new disk with `mkfs.ext4`, mount at `/mnt/data`, stop Docker, move `/var/lib/docker` to new disk, create symlink, restart Docker|✅ Worked — provided 50GB additional space for Docker and Minikube|
|6|**Prometheus/Grafana Helm install timed out** on kube-cloud when using `kube-prometheus-stack`|Switch to the lightweight `prometheus` Helm chart with reduced settings: `server.retention=2h` and `server.global.scrape\_interval=30s`|✅ Worked — significantly reduced resource usage and install time|

\---

## Technical Opinion on the Effectiveness of Using GenAI

Using Microsoft Copilot as a troubleshooting assistant during this project proved to be highly effective, particularly for resolving Kubernetes-specific configuration issues that would otherwise have required extensive manual research across multiple documentation sources and community forums.

**Strengths observed:**

* **Speed:** Copilot provided targeted, command-level solutions almost instantly. Issues that might have taken 30–60 minutes to debug independently were resolved in under 5 minutes in most cases.
* **Accuracy:** The majority of proposed solutions were correct on the first attempt, especially for well-defined errors such as kubeconfig permission issues and Helm cache errors.
* **Contextual awareness:** Copilot was able to interpret screenshots and partial error messages to diagnose root causes effectively, rather than requiring perfectly formatted logs.
* **Adaptability:** When a solution did not work, Copilot suggested alternative approaches without requiring the problem to be re-explained from scratch — for example, switching from VDI resize to adding a second disk when the resize approach failed.

**Limitations observed:**

* **Environment specificity:** Some solutions required adaptation for Ubuntu 24.04, as certain commands differed slightly from Ubuntu 22.04 which is more commonly documented online.
* **No direct system access:** Since Copilot cannot directly inspect the VM state, some back-and-forth was needed to narrow down the exact cause of complex issues such as disk space exhaustion.
* **Partial solutions:** Two of the six documented issues resulted in partial solutions, requiring iterative follow-up prompts to reach full resolution.

**Overall Assessment:**

GenAI assistance significantly accelerated the setup and debugging phases of this project. It functioned best as an interactive technical guide — providing a strong starting point for each issue while still requiring the user to execute, verify, and adapt solutions based on real system feedback. It is not a replacement for understanding the underlying technology, but it is an effective tool for reducing time spent on repetitive or well-known configuration problems.

