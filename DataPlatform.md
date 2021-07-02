# Data Platforms

**produced by Dave Lusty**

## Introduction

This document aims to set out what a data platform is and what is necessary to make it workable. I have tried to minimise complexity here for the sake of clarity, and have tried to keep all terms used as simple as possible. The concepts here align well with the distributed data mesh concepts published elsewhere, the aim here is to explain how you might go about using those concepts in a real world platform. It's important to understand that most of the ideas here require operational change in how the data team works on a day to day basis, and how it interacts with the wider business.

## Data Platform

At it's simplest, a data platform consists of some kind of catalog to find and describe data, and a bunch of data products which will each house some kind of solution, be it a data set or a streaming data analytics solution. Each one is an encapsulated system with known outputs and inputs. I have chosen to avoid going into technical architecture at this stage, because it distracts from the real challenge of a data platform which is how it fits within the organisation, who manages and maintains it, and who builds it? These questions must be answered before you can decide any technical aspects of the design, and they will heavily influence that design. Please bear in mind that this document describes a "cloud" data environment. If you are building a legacy architecture where you deploy the platform and then fill it with data, this document is not for you. This architecture is cloud native from the ground up, meaning that components are deployed as needed and the solution is modular with individually scaled parts. That is not to say this must be public cloud. A data product can, and often will be implemented using traditional on-premises data products. The idea of cloud architecture is related to the way we implement and scale a solution in a more modular way.

<table>
<tr>
<td width="25%">&nbsp;</td>
<td width="50%"><img src="images/DataPlatformOverview.png" alt="Diagram showing the components of a data platform which is made up of multiple data products" /></td>
<td width="25%">&nbsp;</td>
</tr>
</table>

## Data Products

Data products are the basic component of the platform. Each data product is a self contained and self describing product that must form a contract with the outside world and must be discoverable. In an ideal world, a data product should not have any tightly coupled dependencies and while it may depend on other data products, that relationship should be loosely coupled such that, for instance, late processing of one does not affect the other. If tightly coupling is required, for instance to meet processing time requirements, then it is often better to encapsulate the whole process inside a single product. While this might mean duplication of effort in some cases, it is ultimately more manageable in the long term.

<table>
<tr>
<td width="25%">&nbsp;</td>
<td width="50%"><img src="images/DataProduct.png" alt="Diagram showing the various attributes of a data product. These are data lifecycle, access control, infrastructure, owner/steward, data sources, data set lifecycle (project, ITIL, Agile), team, and the data contract. The data contract comprises Data, data format(s), data transformations, availability, schema(s), performance, sensitivity, compliance" /></td>
<td width="25%">&nbsp;</td>
</tr>
</table>

### The Product

We use the word product intentionally here since it's widely used within the technical community and specifically in DevOps to describe an output which is being built by a team. The product will have features, and it will have ownership to guide its development. There will be a feedback loop in place to allow its "customers" to ask for improvements, changes, or new features. Calling it a product instils the idea of a vendor-customer relationship which is useful to understand why the product exists, as well as to understand who is paying for it, either directly or indirectly. If the customer asks for a new feature, the decision must be made as to whether that feature should be included in the current product, or if it's sufficiently different as to require a new product.
In the same way as any other product, data products might be stand alone, or they may consist of components. These components can be other products. For example, a code library developed purely to support the immediate product might just be a part of the project, while a code library that's used across the business would be a product itself, with its own owners, team, requirements etc. Each product will have a deployment cycle to bring it to production, and a clear location from which to consume it. In the case of a data set that might be a data warehouse, a code library used for data quality in multiple data products would use an object repository such as Azure Artifacts, and a machine learning model might be deployed to a container registry ready to be used by other projects.

<table>
<tr>
<td width="25%">&nbsp;</td>
<td width="50%"><img src="images/components.png" alt="Diagram showing how dependencies work between data products. Here, we see three data products each dependent on the last, as well as a code library product being used by the second and third data products" /></td>
<td width="25%">&nbsp;</td>
</tr>
</table>

### The Team

Before creating a data product you need to determine who owns that data product from a "product" point of view. Traditionally, once data was removed from a system of record the ownership transferred to the "data team" whatever form that took. A different way to look at this is to imagine that the owner of the source system is it's own entity and is responsible for sharing that data out with the world. This has many benefits which are rooted in the fact that the team who run the source system already understand the data, the infrastructure, and the usage as well as any number of important aspects such as compliance. Generally the data team will not be aware of these details and so have to learn them during the project, slowing down progress. This is not to say that the team responsible for the SAP ERP system must also be responsible for building the data pipelines to get it into a data lake, but they should certainly play a core role in the team that does so. This draws from the idea of DevOps, where the team is made up of cross discipline members who, together, have the ability to create and run the product. Building such a team can lead to great efficiencies due to familiarity with the domain. 
The team will decide how to present their data product out to the world. This may seem obvious, but it is often ignored in more traditional "layered" approaches where raw extracts become one data product and a cleansed version of that same data become another. With a data centric approach it would be quite normal to only present the cleaned up data to the outside world, even if within the data product infrastructure there exists a copy of the raw extracted data (and usually there would, to allow rebuilding and refactoring).

> **Decision**
>
> Within your architecture and organisation you should decide who will be the "publisher" of data products. Will each team be responsible for presenting their own data (possibly with the help of the data team), or will you centralise that task and have a data team extract data from those systems? In both cases you can still segregate by data product. This decision can be seen as similar to the centralised vs hub and spoke models.

### Product Boundaries

Understanding where your product boundaries lie is important in determining what the scope of a project and team is. As a general rule, your product boundary will be defined either based on the attributes of the product, or by the customers. If you have a single logical customer, and a single set of attributes such as availability and sensitivity then you will probably have a single product. If you're delivering to two separate logical customers, for instance your machine learning model is used by two applications, then your product will probably be separated from those customer systems. If you find that a customer would like access to a presentation interface on one of your components then you may choose to split your product, or just deliver two interfaces. If your two interfaces have different requirements such as SLA, your product will almost certainly need to be split.

<table>
<tr>
<td width="35%">If everything is encapsulated in one product with one interface then you do not have as many interfaces or dependencies, but the product itself becomes more complex to develop and deliver. Your project will have many more tasks and more resources working concurrently, making it harder to manage.</td>
<td width="65%"><img src="images/DataProductBoundary1.png" alt="Diagram of a data product which includes all components from source to reporting including the data warehouse and transformation engine. There is a single presentation interface outside of the product." /></td>
</tr>
</table>

<table>
<tr>
<td width="35%">If someone else in the organisation needs access to another part of your product, for instance some data earlier in the process, then you might need to split the product. This will allow you to maintain two product contracts, and two sets of requirements and requests. It also has the effect of making each project smaller, and separating the delivery lifecycles.</td>
<td width="65%"><img src="images/DataProductBoundary2.png" alt="Diagram of the same components as the previous diagram. This time there is a data product for the data ingest to storage account which has a presentation interface (Blob storage), and a second product which transforms that data and presents in Power BI." /></td>
</tr>
</table>

<table>
<tr>
<td width="35%">Alternatively you may wish to set your products up in a much more granular way, with more available interfaces to allow flexibility of use. While this is conceptually much simpler, it means that every product to product interface is reliant on a contract and so may become harder or slower to make breaking changes. Managing downstream compatibility can also be more challenging in environments built this way.</td>
<td width="65%"><img src="images/DataProductBoundary3.png" alt="Diagram of the same components as the previous diagram. This time there is a data product for the data ingest to storage account which has a presentation interface (Blob storage), and a second product which transforms that data into Synapse data warehouse, which has its own presentation interface, and a third product which takes that model and presents in Power BI." /></td>
</tr>
</table>

<table>
<tr>
<td width="35%">In the machine learning world, a product by default will be the trained ML model and the team might just work on the training aspect of the product. This would leave other projects or products to take the trained model and either use it directly or make a service which presents it out.</td>
<td width="65%"><img src="images/DataProductBoundaryML1.png" alt="Diagram of a machine learning data product where two data products are used to train a model and deploy to a container registry. A second product then uses this container image and deploys to AKS as part of a web app." /></td>
</tr>
</table>

<table>
<tr>
<td width="35%">Embracing the "DevOps" mentality, the operational aspects of consuming the model might be brought into scope for the project. In true cloud style the model might then be provided as SaaS within the business, and the machine learning product team becomes responsible for scaling and managing the solution as other business users consume the model freely.</td>
<td width="65%"><img src="images/DataProductBoundaryML2.png" alt="The same diagram as above, but this time the machine learning data product also publishes the container image as an API which is consumed by the web app as a service" /></td>
</tr>
</table>

<table>
<tr>
<td width="35%">Finally, the machine learning aspects of the product might just be a component within a business application project. In this scenario there is only one consumer of the model and it is not reused elsewhere in the business.</td>
<td width="65%"><img src="images/DataProductBoundaryML3.png" alt="The same diagram as above, but this time the web app is also included within the machine learning product, making the team responsible for the whole end to end application" /></td>
</tr>
</table>

> **Decision**
>
> Within your architecture and organisation you should decide where you will draw product boundaries. This decision will affect both product and operational simplicity. The more you encapsulate within a product, the fewer outside dependencies there will be, and the fewer projects there are to manage. The counterpoint to this is that flexibility and agility are reduced since you will be building larger products with more moving parts.

### Data

#### Sources

Each data product must comprise of zero or more upstream sources for its data. We include zero here because often a lake will include random and manufactured data for testing purposes which will not have a source. In these instances the architecture may look very different for the data product when compared to the standard ELT model.

## Examples

### Ingest Data

<table>
<tr>
<td width="50%">An ingest data product is used for bringing data into the platform. As a general rule data will not be modified within this data product, and the data will be read only once ingested, so as to make an immutable copy which can be used later to reprocess other data products without going back to source systems. These are often owned by the owner of the source system of record, who may occasionally change their own schema and as a result will need to update the data product which consumes it. It is likely that the same team may also create a second data product which is a refined version of this data, ready for consumption by the business.</td>
<td width="50%"><img src="images/examples/dataIngest.png" alt="diagram showing an ingest pattern with database, data factory and storage account" /></td>
</tr>
</table>

### Refine and Enrich Data

<table>
<tr>
<td width="50%">A common scenario is that multiple data sources will be combined to create a new data product, or a master version of a data table. These refinement patterns might also be used to transform and refine a single "raw" data set to make it more consumable by laying out the data in simpler semantic schemas, or translating to one or more consumable file formats (CSV, Parquet, etc.).</td>
<td width="50%"><img src="images/examples/refineAndEnrichDataSet.png" alt="diagram showing an enrichment pattern with multiple data products feeding into data factory and transformed with Databricks and then into a storage account" /></td>
</tr>
</table>

### Reclassify Data

<table>
<tr>
<td width="50%">When anonymising, pseudonymising or otherwise transforming data to be less sensitive for compliance reasons or when doing development work where people have more free access you may need a specific data product to do so, in which case this pattern might be used. More commonly, this task would be undertaken within the source data product, and two almost identical data sets created within a single data product. One data set will being sensitive and the other not.</td>
<td width="50%"><img src="images/examples/reclassify.png" alt="diagram showing a reclassification pattern which takes data from a storage account, processes with data factory and Databricks to remove or replace sensitive data and puts it into a new storage account" /></td>
</tr>
</table>

### Event Data

<table>
<tr>
<td width="50%">Event based data is often not a core part of a data platform, but rather a part of some other near real-time system. The pattern for this shows that there is an event flow with eventual presentation (perhaps within the event app itself) but we also take a copy of the event data onto the lake for consumption by other products.</td>
<td width="50%"><img src="images/examples/eventData.png" alt="diagram of an event data pattern showing an app feeding an Event Hub which dumps data to a storage account as well as passing it to Stream Analytics and on to Power BI" /></td>
</tr>
</table>

### External Data

<table>
<tr>
<td width="50%">Data from sources outside of the business will often have a very different pattern to internal ingestion. This might take the form of a Logic App to integrate with external APIs and request data, or a web hook to allow third parties to push data into the environment. These patterns might include more checks for security or hygiene of the data before presenting it internally as a data product.</td>
<td width="50%"><img src="images/examples/externalData.png" alt="diagram showing an external data pattern where a Logic App gets data from an external API source then puts the data into a storage account" /></td>
</tr>
</table>

### Code Library

<table>
<tr>
<td width="50%">When code libraries are consumed by multiple products they will become products of their own. This pattern shows how the library code is put through a build process to compile (where necessary) and then the built package tested and pushed into an object repository from where it can be consumed. The object repository allows consumers to manage versioning and updating in the same way as public library repositories. While not technically a data product I have included it here because there will often be libraries for data quality checks or cross referencing in a data environment which will produce code packages for internal use.</td>
<td width="50%"><img src="images/examples/codeLibrary.png" alt="diagram of a code library pattern where function code is compiled in a build process and the library is placed into an object store (Azure Artifacts) for use in other products" /></td>
</tr>
</table>

### Modelled Data and EDW

<table>
<tr>
<td width="50%">Enterprise Data Warehouses are a core pattern in a data platform. These will consume other data products in the environment and then model the data into consumable information</td>
<td width="50%"><img src="images/examples/modelledDataEDW.png" alt="diagram of an enterprise data warehouse pattern where data is taken from multiple data products and modelled with Data Factory and Databricks and then placed into storage before loading with Polybase into Synapse DW" /></td>
</tr>
</table>

### Reporting

<table>
<tr>
<td width="50%">Reporting is another common scenario where data products are consumed and pushed into a reporting platform for consumption or alongside pre-canned reports. Often for heavily utilised reporting, a caching layer will be used such as Analysis Services to enable much higher concurrency, add a role based security layer, or simply add a semantic layer to the data.</td>
<td width="50%"><img src="images/examples/reporting.png" alt="diagram of a reporting pattern showing a Synapse data warehouse feeding Azure Analysis Services and Power BI" /></td>
</tr>
</table>

### Machine Learning

<table>
<tr>
<td width="50%">Machine learning is a common scenario where data products, and often other data sources, are used to train a machine learning model. MLOps will potentially then be used to train and test this model before delivering it to a container registry where it can be consumed by other services. The other services will then use the container registry to version control the model they use to process data, and they will pull that container image to create a service which processes data in their own product.</td>
<td width="50%"><img src="images/examples/machineLearning.png" alt="diagram of a machine learning pattern where data products feed a training process in Azure ML before being published into a container registry" /></td>
</tr>
</table>