# Deployment

**Persona needed**: Devops Engineer

For this guide, we will deploy the Digital human avatar and the Retrieval Augmented Generation pipeline on separate instances. 
NOTE:  Here detail a way to deploy the digital human pipeline agnostic of a cloud provider. The deployment specific to a cloud provider can be found [here](https://docs.nvidia.com/ace/latest/workflows/tokkio/text/deployment/AWS_CSP_Setup_Guide_automated.html). 

The digital avatar deployment uses cloud native stack deployment script that automates and abstracts out complexities setup and deployment of our application on a virtual or a physical machine with a GPU.  


## Digital Human Pipeline Deployment

For this workflow, we will be leveraging [NVIDIA ACE](https://developer.nvidia.com/ace) - a suite of technologies for bringing digital humans to life with generative AI.

Follow the setup guide [here](https://docs.nvidia.com/ace/latest/workflows/tokkio/text/deployment/Bare_Metal_Setup_Guide_automated.html) to deploy the out of box **Ben** digital human on your virtual or physical machine.

## RAG Pipeline Deployment

Follow the steps [here](https://github.com/NVIDIA/GenerativeAIExamples/tree/main/RAG/examples/basic_rag/langchain) to set up the RAG pipeline on a different instance.

Note: For deploying RAG with customization, please take a look at the customization(../customize/) section of this guide

## Connect your digital human avatar to domain adapted RAG

Now that you have deployed the digital avatar and the Retrieval-Augmented Generation (RAG) application, the next step is to connect the two pipelines so that they can communicate with one another using the REST API. To do this, we will point the Digital Avatar application to our RAG Server endpoint by doing a helm upgrade.  To do this, we will first pull a helm to update the values.yaml.

From the previous section, we can have our ORAN RAG Server running at `http://localhost:8081` or if using an AWS EC2 instance to deploy it can be running somewhere like `http://52.39.xx.xx:8081`

With your baseline Avatar UI setup running on the AWS instance and functioning properly, we now need to point the app to our RAG Server endpoint by doing a `helm upgrade`.

### 1. **Fetch the Helm Chart**:
On the AWS Instance, first verify that all the pods are up and running.
```
kubectl get pods
```
We can then fetch the `ucs-tokkio-audio-video-llm-app` from NVIDIA GPU Cloud.
```bash
helm fetch https://helm.ngc.nvidia.com/nvidia/ace/charts/ucs-tokkio-app-base-3-stream-llm-rag-3d-ov-4.1.3.tgz --username='$oauthtoken' --password=NGC_API_KEY
```
### 2. **Extract the Chart Package**:
```bash
tar -xzf ucs-tokkio-app-base-3-stream-llm-rag-3d-ov-4.1.3.tgz

#Files inside the app
cd ucs-tokkio-app-base-3-stream-llm-rag-3d-ov/
ls -l
    app_info.yaml
    Chart.yaml
    values.yaml
  
```

### 3. **Edit the Values File to add RAG Endpoint**:

Make a copy of the `values.yaml` > `new_values.yaml`

```bash
cp values.yaml new_values.yaml

#make edits to the file
nano new_values.yaml
```

```
ace-agent-plugin-server:
pluginConfig:
   plugins:
      rag:
      parameters:
         USE_RAG: false
         RAG_SERVER_URL: http://IP:PORT
         NIM_MODEL: "meta/llama3-8b-instruct" #as defined at https://build.nvidia.com/meta/llama-3_1-8b-instruct
         USE_OPENAI: false
         OPENAI_MODEL: "gpt-4"
```

In the `new_values.yaml` file, update the `RAG_SERVER_URL` under `ace-agent-plugin-server` section:


### 4. **Upgrade the Helm Release**:
Apply the changes to reflect in your already deployed helm chart by upgrading the Helm release with the `new_values.yaml` file.

```bash
helm upgrade --values <path_to_new_values.yaml> <release_name> <chart_name>
```
You can get the chart name and release name by using `helm ls` on your instance. 

Example helm upgrade command:

```bash
helm upgrade --values ucs-tokkio-app-base-3-stream-llm-rag-3d-ov/new_values.yaml tokkio-app ucs-tokkio-audio-video-llm-app
```

### 5. **Verify the Deployment**:
Check that all pods are up and running to ensure the deployment was successful.

```bash
ubuntu@ip-10-0-0-135:~$  kubectl get pods
NAME                                                        READY   STATUS    RESTARTS   AGE
a2f-a2f-deployment-6d9f4d6ddd-n6gc9                         1/1     Running   0          71m
ace-agent-chat-controller-deployment-0                      1/1     Running   0          71m
ace-agent-chat-engine-deployment-687b4868c-dx7z8            1/1     Running   0          71m
ace-agent-plugin-server-deployment-7f7b7f848f-58l9z         1/1     Running   0          71m
anim-graph-sdr-envoy-sdr-deployment-5c9c8d58c6-dh7qx        3/3     Running   0          71m
chat-controller-sdr-envoy-sdr-deployment-77975fc6bf-tw9b4   3/3     Running   0          71m
ds-sdr-envoy-sdr-deployment-79676f5775-64knd                3/3     Running   0          71m
ds-visionai-ds-visionai-deployment-0                        2/2     Running   0          71m
ia-animation-graph-microservice-deployment-0                1/1     Running   0          71m
ia-omniverse-renderer-microservice-deployment-0             1/1     Running   0          71m
ia-omniverse-renderer-microservice-deployment-1             1/1     Running   0          71m
ia-omniverse-renderer-microservice-deployment-2             1/1     Running   0          71m
mongodb-mongodb-bc489b954-jw45w                             1/1     Running   0          71m
occupancy-alerts-api-app-cfb94cb7b-lnnff                    1/1     Running   0          71m
occupancy-alerts-app-5b97f578d8-wtjws                       1/1     Running   0          71m
redis-redis-5cb5cb8596-gncxk                                1/1     Running   0          71m
redis-timeseries-redis-timeseries-55d476db56-2shpt          1/1     Running   0          71m
renderer-sdr-envoy-sdr-deployment-5d4d99c778-qm8sz          3/3     Running   0          71m
riva-speech-57dbbc9dbf-dmzpp                                1/1     Running   0          71m
tokkio-cart-manager-deployment-55476f746b-7xbrg             1/1     Running   0          71m
tokkio-ingress-mgr-deployment-7cc446758f-bz6kz              3/3     Running   0          71m
tokkio-menu-api-deployment-748c8c6574-z8jdz                 1/1     Running   0          71m
tokkio-ui-server-deployment-55fcbdd9f4-qwmtw                1/1     Running   0          71m
tokkio-umim-action-server-deployment-74977db6d6-sp682       1/1     Running   0          71m
triton0-766cdf66b8-6dsmq                                    1/1     Running   0          71m
vms-vms-bc7455786-6w7cz                                     1/1     Running   0          71m
```

### 6. **Spin up the UI**:
Navigate to the UI using the URL provided in the deployment output to interact with the updated Tokkio-ORAN Digital Human Avatar.

`https://<ui_sub_domain>.<base_domain>`

You're ready to ask questions to your Avatar!
