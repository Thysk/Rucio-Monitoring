# Rucio Database based monitoring

Database based monitoring uses logstash to schedule, make queries from the Rucio database and send the results to Elastic/OpenSearch to be parsed for Visualisation.

Initial pipelines were developed by Li Teng and modified by Timothy Noble to be used in Rucio version 34 and created an easy to deploy version using helm and kustomise.

Within this directory are the resources to setup and use a logstash to deploy some database querying pipelines as the inputs and send the information to a XXXXSearch deployment as the output.

In brief. the deployment process goes as follows:
1. Edit the helm chart dependant on which Search software you have deployed
2. Deploy secrets to the cluster to be used by logstash to query the database and write to Search
3. Deploy Logstash using the helm chart
4. Register the various indexes as datasources in Grafana
5. Deploy dashboards to Grafana to visualise the data

## 1. Edit the helm chart dependant on which Search software you have deployed

## 2. Deploy secrets to the cluster to be used by logstash to query the database and write to Search

## 3. Deploy Logstash using the helm chart

## 4. Register the various indexes as datasources in Grafana

## 5. Deploy dashboards to Grafana to visualise the data
