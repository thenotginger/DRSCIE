# Deployment

**Persona needed**: Devops Engineer

For this guide, we will deploy the Digital human avatar and the Retrieval Augmented Generation pipeline on separate instances. 
NOTE:  Here detail a way to deploy the digital human pipeline agnostic of a cloud provider. The deployment specific to a cloud provider can be found [here](https://docs.nvidia.com/ace/latest/workflows/tokkio/text/deployment/Deployment.html). 

The digital avatar deployment uses cloud native stack deployment script that automates and abstracts out complexities setup and deployment of our application on a virtual or a physical machine with a GPU.  


## Digital Human Pipeline Deployment

For this workflow, we will be leveraging [NVIDIA ACE](https://developer.nvidia.com/ace) - a suite of technologies for bringing digital humans to life with generative AI.

Follow the quickstart guide [here](https://docs.nvidia.com/ace/latest/workflows/tokkio/text/Quickstart_Guide.html) to deploy the out of box **Taylor** digital human on your virtual or physical machine.

The Digital Human Pipeline broadly involves four components to be installed:
1. The Tokkio Backend: This is the core of the Digital Human pipeline and is a helm chart that comprises of all the inference technologies and orchestration pipeline and is deployed as a helm chart on Kubernetes environment.
2. The Media Relay Service: This component is responsible for relaying media between Client browser and Tokkio Back-end.
3. The Tokkio UI Clinet / Tokkio Front End: A Web application front-end comprising necessary components to interact with Tokkio Back-end.
Read more about each component and the deployment architecture [here](https://docs.nvidia.com/ace/latest/workflows/tokkio/text/deployment/Overview.html)

## RAG Pipeline Deployment

Follow the steps [here](https://github.com/NVIDIA/GenerativeAIExamples/tree/main/RAG/examples/basic_rag/langchain) to set up the RAG pipeline on a different instance.

Note: For connecting the DH pipeline to RAG, please take a look at the customization(../customize/) section of this guide
