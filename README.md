# kubearmor-elasticstack-logging


## E(F/L)K (Elasticsearch, Fluentd/Logstash and Kibana) [WIP]

<img src="./assets/ka-efk.jpg" alt="example image" width="900" height="600">


There would be 3 components in this setup:

1. **Elasticsearch** is a real-time, distributed, and scalable search engine which allows for full-text and structured search, as well as analytics. Relay server logs can be indexed and searched through which would be produced in large volumes of log data.

2. **Kibana** is a data visualization frontend and dashboard for Elasticsearch. Kibana allows user to explore the log data in a visual manner that is stored in the Elasticsearch instance with the help of a web interface. Users would also be allowed to build dashboards or view existing ones which would help to answer and quickly gain insight about the pods managed by KubeArmor:

- Security policies deployed for their pods
- Logs for each policy in a pod
- Logs filtered by severity
- Logs filtered by type (MatchedPolicy, MatchedHostPolicy, MatchedNativePolicy)
- Logs filtered by operation (Process, File, Network, Capabilities)
- Logs filtered by action (Allow, Audit, Block)

Kibana and ElastiSearch will be a part of StatefulSet that can run in any node

3. **FluentD**

Why preferred over Logstash?

- It is more memory-efficient.
- Fluentd has a better routing approach, as tagging events is easier than using if-else conditions.

How FluentD could be used:

1. Directly accessing the logs through stdout in Relay Server. It would be done with fluentd plugins made to consume logs from pods eg ```kubernetes_metadata```
OR
2. Using the `kubearmor.kube-system.svc` based on the gRPC service here: https://github.com/kubearmor/kubearmor-client/tree/main/log and then sending over to fluentd

How to deploy FluentD:

Daemonset - There is official support for that from FluentD at https://github.com/fluent/fluentd-kubernetes-daemonset

or

Deployment - As relay is not deployed on every node , we can use Deployment to connect to relays grpc service 

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
value: "elasticsearch.default.svc.cluster.local"
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

In this case fluentd-image uses plugins like ```kubernetes_metadata``` and ```elasticsearch``` to communicate with kubearmor relay and elastisearch instance 

TODO: Currently fluentd-image is configured to ingest all logs generated from kubernetes objects . For kube-armor logging the configuration should filter out only logs from relay pod 

Making a Operator (Using KubeBuilder SDK):

As there are many components involved, a way to manage them could be through an Operator.

Why Operator instead of StatefulSet?

1. Custom Logic: For example, an Elasticsearch operator might perform rolling updates, manage backup and restore operations, and handle scaling of the Elasticsearch cluster.

2. Automation: An Elasticsearch operator might automatically handle node failures and perform self-healing operations to ensure the cluster is always available.

3. Portability: An operator can be used to deploy and manage your application across different Kubernetes clusters%
