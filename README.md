# kubearmor-elasticstack-logging


## E(F/L)K (Elasticsearch, Fluentd/Logstash and Kibana) [WIP]

<img src="./assets/ka-efk.jpg" alt="example image" width="900" height="600">


There would be 3 components in this setup:

1. **Elasticsearch** is a real-time, distributed, and scalable search engine which allows for full-text and structured search, as well as analytics. Relay server data can be indexed and searched through which would be produced in large volumes of log data.

2. **Kibana** is a data visualization frontend and dashboard for Elasticsearch. Kibana allows user to explore the log data in a visual manner that is stored in the Elasticsearch instance with the help of a web interface. Users would also be allowed to build dashboards or view existing ones which would help to answer the quickly gain insight about the pods managed by KubeArmor:

- Security policies deployed for their pods
- Alerts for each policy in a pod
- Alerts filtered by severity
- Alerts filtered by type (MatchedPolicy, MatchedHostPolicy, MatchedNativePolicy)
- Alerts filtered by operation (Process, File, Network, Capabilities)
- Alerts filtered by action (Allow, Audit, Block)

Kibana and ElastiSearch can be part of a Stateful set that runs in the node where grp

3. **FluentD**

Why preferred over Logstash?

- It is more memory-efficient.
- Fluentd has a better routing approach, as tagging events is easier than using if-else conditions.

How FluentD could be used:

1. Directly accessing the logs through stdout in Relay Server. It can be done by using client-go library or running proxy for kube-apiserver for KubeArmor to access the Kubernetes API.

2. Using the `kubearmor.kube-system.svc` based on the gRPC service here: https://github.com/kubearmor/kubearmor-client/tree/main/log

How to deploy FluentD:

Daemonset - There is official support for that from FluentD at https://github.com/fluent/fluentd-kubernetes-daemonset

Example:

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
name: fluentd
namespace: kube-system
...
spec:
...
spec:
containers:
- name: fluentd
image: quay.io/fluent/fluentd-kubernetes-daemonset
env:
- name:  FLUENT_ELASTICSEARCH_HOST
value: "elasticsearch-logging"
- name:  FLUENT_ELASTICSEARCH_PORT
value: "9200"
...
```

TODO: Configure it to deploy in nodes where Relay server is deployed.

To store the configuration for FluentD, we can store as ConfigMaps.

Configuration would require:

- `source` directives determine the input sources.
- `match` directives determine the output destinations.
- `filter` directives determine the event processing pipelines.

Making a Operator (Possibly using KubeBuilder SDK):

As there are many components involved, a way to manage them could be through an Operator.

Why Operator instead of StatefulSet?

1. Custom Logic: For example, an Elasticsearch operator might perform rolling updates, manage backup and restore operations, and handle scaling of the Elasticsearch cluster.

2. Automation: An Elasticsearch operator might automatically handle node failures and perform self-healing operations to ensure the cluster is always available.

3. Portability: An operator can be used to deploy and manage your application across different Kubernetes clusters%
