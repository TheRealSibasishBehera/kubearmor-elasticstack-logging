Open the Kibana UI using portforward   
```kubectl port-forward deployment/kibana 5601:5601```   
Open ```localhost:5601``` in your browser  and head to the Management Console inside it.

![Kibana Management Console](../assets/kibana-management.webp)

Step 2: Select the “Index Patterns” option under Kibana section.

![Create Kubernetes Fluentd Index Pattern](../assets/kibana-index-pattern.webp)

Step 3: Create a new Index Pattern using the pattern – “logstash-*”.

![Kibana Create Kubernetes Fluentd Index](../assets/kibana-index-logstash.png)

Step 4: Select “@timestamp” in the timestamps option.

![Kibana Select Timestamp](../assets/kibana-time.png)

Expected Dashboard 

![Expected Dashboard ](../assets/kibana-dash.png)