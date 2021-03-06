When organizations collaborate, they share information. But most parties don't want to give other parties access to all parts of the data. Mechanisms exist for safeguarding data at rest and in transit. However, encrypting data in use poses different challenges. This article presents a solution that Azure confidential computing (ACC) offers for encrypting in-use data.

By using confidential computing and containers, the solution provides a way for a provider-hosted application to securely collaborate with a hospital and a third-party diagnostic provider. Azure Kubernetes Service (AKS) hosts confidential computing nodes. Azure Attestation establishes trust with the diagnostic provider. By using these Azure components, the architecture isolates the sensitive data of the hospital patients while the specific shared data is being processed in the cloud. The hospital data is then inaccessible to the diagnostic provider. Through this architecture, the provider-hosted application can also take advantage of advanced analytics. The diagnostic provider makes these analytics available as confidential computing services of machine learning (ML) applications.

## Potential use cases

Many industries protect their data by using confidential computing for these purposes:

- Securing financial data
- Protecting patient information
- Running ML processes on sensitive information
- Performing algorithms on encrypted datasets from many sources
- Protecting container data and code integrity

## Architecture

:::image type="complex" source="./media/healthcare-demo-architecture.png" alt-text="Diagram of a confidential healthcare platform demonstration. The platform includes a hospital, medical platform provider, and diagnostic provider." lightbox="./media/healthcare-demo-architecture-enlarged.png":::
Diagram showing how data flows between three parties in a healthcare setting. Three rectangles represent the three parties: a hospital, a medical platform, and a diagnostic provider. Each rectangle contains icons that represent various components, such as a website, a client application, Azure Attestation, a web API, data storage, and a runtime. The medical platform and diagnostic provider rectangles also contain smaller rectangles that represent confidential nodes and A K S clusters. Arrows connect these components and show the flow of data. Numbered callouts correspond to the steps that this article describes after the diagram.
:::image-end:::

The diagram outlines the architecture. Throughout the system:

- Network communication is TLS encrypted in transit.
- Azure Monitor tracks component performance, and Azure Container Registry (ACR) manages the solution's containers.

The solution involves the following steps:

1. A clerk for Lamna Hospital opens the hospital's web portal, an Azure Blob Storage static website, to enter patient data.

1. The clerk enters data into the hospital's web portal, which Contoso Medical Platform Ltd. powers with a Python Flask–based web API. A confidential node in the [SCONE](https://sconedocs.github.io/#scone-executive-summary) confidential computing software protects the patient data. SCONE works within an AKS cluster that has the Software Guard Extensions (SGX) enabled that help run the container in an enclave.

1. The Python code is wrapped in SCONE SGX software. The solution deploys that code on an AKS confidential node as a confidential container. The code stores the data in memory within a Redis cache (**3a**), using Azure Attestation to establish trust (**3b**).

1. If the code doesn't find the patient data, it prompts the clerk to enter the sensitive information on a form. The code then stores that information in Redis.

1. The Contoso application sends the protected patient data from its AKS cluster to an enclave in the Open Neural Network Exchange (ONNX) runtime server. Fabrikam Diagnostic Provider hosts this confidential inferencing server in a confidential node of the Fabrikam AKS cluster.

1. The Fabrikam machine learning–based application obtains the diagnostic results from the confidential inferencing ONNX runtime server. The app sends these results back to the confidential node in the Contoso AKS cluster.

1. The Contoso application sends the diagnostic results from the confidential node back to Lamna's client application.

### Components

- [Static website hosting in Blob Storage](/azure/storage/blobs/storage-blob-static-website) serves static content like HTML, CSS, JavaScript, and image files directly from a storage container.

- [Azure Attestation](/azure/attestation/) is a unified solution that remotely verifies the trustworthiness of a platform. Azure Attestation also remotely verifies the integrity of the binaries that run in the platform. Use Azure Attestation to establish trust with the confidential application.

- [AKS Cluster](/azure/aks/intro-kubernetes) simplifies the process of deploying a Kubernetes cluster.

- [Confidential computing nodes](/azure/confidential-computing/confidential-nodes-aks-overview) are hosted on a specific virtual machine series that can run sensitive workloads on AKS within a hardware-based trusted execution environment (TEE) by allowing user-level code to allocate private regions of memory, known as enclaves. Confidential computing nodes can support confidential containers or enclave-aware containers.

- [SCONE platform](https://azuremarketplace.microsoft.com/marketplace/apps/scontainug1595751515785.scone?tab=Overview) is an Azure Partner independent software vendor (ISV) solution from Scontain.

- [Redis](https://redis.io/) is an open-source, in-memory data structure store.

- [Secure Container Environment (SCONE)](https://sconedocs.github.io/) supports the execution of confidential applications in containers that run inside a Kubernetes cluster.

- [Confidential Inferencing ONNX Runtime Server Enclave (ONNX RT - Enclave)](https://github.com/microsoft/onnx-server-openenclave) is a host that restricts the ML hosting party from accessing both the inferencing request and its corresponding response.

- [Azure Monitor](/azure/azure-monitor/overview) collects and analyzes app telemetry, such as performance metrics and activity logs.

- [ACR](https://azure.microsoft.com/services/container-registry/) is a service that creates a managed registry. ACR builds, stores, and manages container images and can store containerized machine learning models.

### Alternatives

- You can use [Fortanix](https://www.fortanix.com) instead of SCONE to deploy confidential containers to use with your containerized application. Fortanix provides the flexibility you need to run and manage the broadest set of applications: existing applications, new enclave-native applications, and pre-packaged applications.

- [Graphene](https://graphene.readthedocs.io/en/latest/cloud-deployment.html#azure-kubernetes-service-aks) is a lightweight, open-source guest OS. Graphene can run a single Linux application in an isolated environment with benefits comparable to running a complete OS. It has good tooling support for converting existing Docker container applications to Graphene Shielded Containers (GSC).

## Considerations

Azure confidential computing virtual machines (VMs) are available in 2nd-generation D family sizes for general purpose needs. These sizes are known collectively as D-Series v2 or DCsv2 series. This scenario uses Intel SGX-enabled DCs_v2-series virtual machines with Gen2 operating system (OS) images. But you can only deploy certain sizes in certain regions. For more information, see [Quickstart: Deploy an Azure Confidential Computing VM in the Marketplace](/azure/confidential-computing/quick-create-marketplace) and [Products available by region](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines).

## Deploy this scenario

Deploying this scenario involves the following high-level steps:

- Deploy the confidential inferencing server on an existing SGX-enabled AKS Cluster. See the [confidential ONNX inference server](https://github.com/microsoft/onnx-server-openenclave) project on GitHub for information on this step.

- Configure Azure Attestation policies.

- Deploy an SGX-enabled AKS cluster node pool.

- Get access to [curated confidential applications called SconeApps](https://sconedocs.github.io/helm/). SconeApps are available on a private GitHub repository that's currently only available for commercial customers, through SCONE Standard Edition. Go to the [SCONE website](https://scontain.com/) and contact the company directly to get this service level.

- Install and run SCONE services on your AKS cluster.

- Install and test the Flask-based application on your AKS cluster.

- Deploy and access the web client.

These steps focus on the enclave containers. A secured infrastructure would extend beyond this implementation and include compliance requirements, such as added protections required by HIPAA.

## Pricing

To explore the cost of running this scenario, use the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator), which preconfigures all Azure services.

A [sample cost profile](https://azure.com/e/5e776a5dbebf4f20974ebbfa0e247747) is available for the Contoso Medical SaaS Platform, as pictured in the diagram. It includes the following components:

- System node pool and SGX node pool: no disks, all ephemeral
- AKS Load Balancer
- Azure Virtual Network: nominal
- Azure Container Registry
- Storage account for single-page application (SPA)

The profile doesn't include the following components:

- Azure Attestation Service: free
- Azure Monitor Logs: usage based
- SCONE ISV licensing
- Compliance services required for solutions working with sensitive data, including:

  - Azure Security Center and Azure Defender for Kubernetes
  - Azure DDoS Protection: standard
  - Azure Firewall
  - Azure Application Gateway and Azure Web Application Firewall

## Next steps

- Learn more about [Azure confidential computing](/azure/confidential-computing/).
- See the [confidential ONNX inference server](https://github.com/microsoft/onnx-server-openenclave) project on GitHub.

## Related resources

- [Confidential containers on AKS](/azure/confidential-computing/confidential-containers).
- [Official ONNX runtime website](https://www.onnxruntime.ai/).
- [Confidential ONNX inference server (GitHub sample)](https://github.com/microsoft/onnx-server-openenclave).
- [MobileCoin use case with anonymized blockchain data](https://customers.microsoft.com/story/844245-mobilecoin-banking-and-capital-markets-azure).
- [A sample brain segmentation image](https://github.com/mateuszbuda/brain-segmentation-pytorch/blob/master/assets/TCGA_CS_4944.png) for use with the delineation function that invokes the confidential inferencing server.