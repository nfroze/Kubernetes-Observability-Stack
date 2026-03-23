# Kubernetes Observability Stack

A production-style observability platform on AWS EKS that combines metrics monitoring and centralized log aggregation, giving teams a single pane of glass across application health and container logs.

## Overview

In any Kubernetes environment beyond a handful of pods, operators lose visibility fast. Logs scatter across nodes, metrics go uncollected, and debugging production issues turns into guesswork. This project builds the observability layer that solves that — a fully instrumented EKS cluster running Prometheus and Grafana for metrics alongside the ELK stack and Fluentd for centralized logging.

The infrastructure is provisioned entirely through Terraform, creating a VPC with public and private subnets, NAT gateway, and an EKS cluster with managed node groups. On top of that, two Kubernetes namespaces separate concerns: `monitoring` runs the Prometheus and Grafana stack, while `logging` runs Elasticsearch, Logstash, Kibana, and Fluentd as a DaemonSet that tails container logs from every node and ships them to Elasticsearch in near real-time.

A sample Nginx application generates the traffic and logs that flow through both pipelines, demonstrating the full observability loop from application event to dashboard.

## Architecture

![](screenshots/cloud-architecture.png)

The system runs inside a VPC with two availability zones. EKS worker nodes sit in private subnets with outbound access through a NAT gateway, while LoadBalancer services expose Grafana, Kibana, and Prometheus for operator access.

The monitoring pipeline flows from Prometheus scraping pod and node metrics on a pull model, with Grafana querying Prometheus as its datasource for dashboards. The logging pipeline flows from Fluentd DaemonSets tailing `/var/log/containers` on each node, enriching logs with Kubernetes metadata, and forwarding them to Elasticsearch where Kibana provides search and visualization. Logstash sits alongside as a parallel ingestion path accepting beats input on port 5044.

## Tech Stack

**Infrastructure**: AWS VPC, EKS, EC2 (t3.medium managed node group), NAT Gateway, Terraform

**Monitoring**: Prometheus (kube-prometheus-stack), Grafana

**Logging**: Elasticsearch, Logstash, Kibana, Fluentd (DaemonSet with Kubernetes metadata enrichment)

**Orchestration**: Kubernetes 1.29, Helm

## Key Decisions

- **Fluentd DaemonSet over sidecar pattern**: DaemonSets collect logs from all containers on a node without requiring application changes. This mirrors how production teams instrument logging at scale — one agent per node rather than one per pod.

- **Separate monitoring and logging namespaces**: Isolating observability workloads by concern makes RBAC scoping cleaner and prevents resource contention between the metrics and logging pipelines.

- **Single NAT gateway in dev, multi in prod**: The Terraform configuration conditionally deploys one or multiple NAT gateways based on environment, balancing cost in dev against high availability in production.

- **LoadBalancer services for observability tools**: Exposing Grafana, Kibana, and Prometheus via AWS LoadBalancers provides direct operator access without requiring kubectl port-forwarding, reflecting how teams access dashboards in real environments.

## Screenshots

![](screenshots/terraform-apply.png)

![](screenshots/eks-cluster.png)

![](screenshots/all-pods-running.png)

![](screenshots/prometheus-targets.png)

![](screenshots/grafana-dashboard.png)

![](screenshots/kibana-homepage.png)

![](screenshots/kibana-discover-logs.png)

![](screenshots/git-branches.png)

## Author

**Noah Frost**

- Website: [noahfrost.co.uk](https://noahfrost.co.uk)
- GitHub: [github.com/nfroze](https://github.com/nfroze)
- LinkedIn: [linkedin.com/in/nfroze](https://linkedin.com/in/nfroze)
