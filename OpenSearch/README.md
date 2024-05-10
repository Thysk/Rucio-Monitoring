# OpenSearch monitoring

Within this directory are the resources to setup and use an OpenSearch deployment, the instructions on how to connect Rucio to said OpenSearch deployment, and how to register the OpenSearch deployment as a datasource in grafana.

In brief, the deployment process goes as follows:
1. Use the helm chart to deploy an instance of OpenSearch
2. Add the index to OpenSearch to accept the messages from Hermes
3. Configure Hermes to have OpenSearch as its elastic_endpoint
4. Register OpenSearch as a datasource in Grafana
5. Deploy dashboards to Grafana to visualise the data


## 1. Use the helm chart to deploy an instance of OpenSearch

There are a number of different ways to utilise the charts. There is utilise a CI/CD pipeline to read the values or helm files and deploy accordingly, Modify the Makefiles to create the helm files and deploy the charts using helm.
Whichever you choose, the default settings for OpenSearch are to have 3 master Nodes and 3 worker nodes.

### A. Using a CI/CD pipeline


### B. Using the Makefiles


## 2. Add the index to OpenSearch to accept the messages from Hermes

Adding the index to the OpenSearch can be completed in two ways: using curl, and the OpenSearch CLI. Here I will cover using curl, adding indexes using the OpenSearch CLI can be found in their [documentation](https://opensearch.org/docs/latest/api-reference/index-apis/create-index/).

### A. Utilising curl

#### Get access to OpenSearch:

Then portforward to the service to get access to the deployment

`kubectl port-forward -n opensearch-system svc/opensearch-cluster-worker 9200`

In another terminal run, the default username and password is admin - this of course should be changed:

`curl -u "<opensearch username>:<OpenSearch password>" -k "https://localhost:9200"`

Should all be correct you will get a response that shows:


>    {  
>    "name" : "opensearch-cluster-master-0",  
>    "cluster_name" : "opensearch-cluster",  
>    "cluster_uuid" : "EmhtKET_SLGrMEi8cSYdPw",  
>    "version" : {  
>        "distribution" : "opensearch",  
>        "number" : "2.8.0",  
>        "build_type" : "tar",  
>        "build_hash" : "db90a415ff2fd428b4f7b3f800a51dc229287cb4",  
>        "build_date" : "2023-06-03T06:24:25.112415503Z",  
>        "build_snapshot" : false,  
>        "lucene_version" : "9.6.0",  
>        "minimum_wire_compatibility_version" : "7.10.0",  
>        "minimum_index_compatibility_version" : "7.0.0"  
>    },  
>    "tagline" : "The OpenSearch Project: https://opensearch.org/"  
>    }  

#### Create index in OpenSearch

Within the Repo from step 1 at overlays/Elastic/OpenSearch/rucio-events contains the index, this needs to be applied to OpenSearch in the location you want the index to be created 

`curl -u "<username>:<password>" -XPUT "https://localhost:9200/<Your chosen index>/?&pretty" -H "Content-Type: application`


## 3. Configure Hermes to have OpenSearch as its elastic_endpoint

OpenSearch is now setup to receive messages from Rucio. It now needs to have Rucio Hermes daemon pointed at it. For this editing of the Rucio values will allow for such thing. 

You will need at least this in the config section of the daemon values 



>  hermes:  
>    services_list: "elastic"  
>    elastic_endpoint: https://opensearch-cluster-worker.opensearch-system.svc.cluster.local:9200/<your chosen index>/_bulk"  

You will also need to create a secret in the Rucio namespace to allow the injection of the OpenSearch user secret for Hermes to be able to deposit the messages 



`kubectl create secret generic <daemons deploymentname>-open-secrets --namespace <Rucio namespace> --from-literal=USERNAME=<USERNAME>--from-literal=PASSWORD=<OpenSearch password>`

Add the secret to Hermes, an example is below

>hermes:  
>  threads: 1  
>  podAnnotations: {}  
>  bulk: 1000  
>  resources:  
>    limits:  
>      memory: "600Mi"  
>      cpu: "210m"  
>    requests:  
>      memory: "300Mi"  
>      cpu: "140m"  
>  - name: RUCIO_CFG_HERMES_ELASTIC_USERNAME  
>    valueFrom:  
>      secretKeyRef:  
>        name: open-secrets  
>        key: USERNAME  
>  - name: RUCIO_CFG_HERMES_ELASTIC_PASSWORD  
>    valueFrom:  
>      secretKeyRef:  
>        name: open-secrets  
>        key: PASSWORD  


## 4. Register OpenSearch as a datasource in Grafana

Now that Rucio is now communicating its logs to OpenSearch it now needs to be able to be visualised using Grafana

In your clusters Grafana deployment go to

- configuration / datasources

- Add data source

- Select OpenSearch

- Install plugin if needed

- Put in the following details in the fields: 

>URL: http://OpenSearch-eck-OpenSearch-es-worker.elastic-system.svc.cluster.local:9200  
>Basic auth: True  
>User: <USERNAME>  
>Password: <OpenSearch password>  
>Index name: <your Index>  
>Pattern: No pattern  
>Time field name: create_at  
>Max concurrent Shard Requests: 3 # If you have used the base repo settings  


## 5. Deploy dashboards to Grafana to visualise the data

Once the data source is created and verified you can create a dashboard using it

Navigate to Dashboards / Import

Paste in the contents of overlays/Elastic/OpenSearch/Dashboards/Rucio External into the import via panel json