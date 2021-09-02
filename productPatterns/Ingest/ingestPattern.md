# Ingest Pattern

**produced by Dave Lusty**

## Introduction

This data ingest pattern can be used to batch ingest data and produce one or more data sets on the data lake.

<table>
<tr>
<td width="15%">&nbsp;</td>
<td width="70%"><img src="images/dataIngest.png" alt="Diagram showing the basic data ingest pattern" /></td>
<td width="15%">&nbsp;</td>
</tr>
</table>

## Service

This section describes the product from an SOA perspective. How will customers consume, request, pay for, etc. the service? What are they requesting, is it a data set, an environment, access to an API?

### Deliverable(s)

This product will deliver a set of data with a specified schema.

### Request mechanism

* An ITSM request will be submitted requesting read only access to the data set.
* An approval process will check the request.
* Time limited credentials and data location will be provided in the response.

## Product

### Inputs

### Interfaces

What interfaces will the product offer? What is the presentation layer/output/deliverable for the product?

### Output registration

Will this product be registered somewhere to make it discoverable? If so, what service will it be registered in?

## Lifecycles

This section details the various lifecycles which make up the data ingest pattern.

### Development Environment Infrastructure

This lifecycle is used to deploy and manage the development environment which will be used to create data pipelines. 

Since this pattern is centred around Azure Data Factory the requirements are largely driven by how that product works. In this instance, the development environment is also responsible for building the deployable asset. This means that the development environment is always different to that of the later staging areas. For this reason the mantra that all environments must be identical with the same assets deployed cannot work without significant effort. For this reason, we have separated out the lifecycle of the development environment. This allows this environment to be redeployed or destroyed as necessary.

<table>
<tr>
<td width="15%">&nbsp;</td>
<td width="70%"><img src="images/lifecycleDev.png" alt="Flow of development infrastructure lifecycle as described in the text below" /></td>
<td width="15%">&nbsp;</td>
</tr>
</table>

#### Stage - Development

* ARM Templates will be written in Visual Studio/VSCode (or some other tool)
* Templates will be committed to the repository in a specific folder under the feature branch
* Manual deployment testing will be done with the ARM templates
* Pull request from feature branch to collaboration branch
* Approval of the pull request will trigger the build process
  * Build process copies ARM templates to create an artefact

#### Stage - Testing

* Artefact is deployed into a testing environment
  * Automated/manual tests confirm that infrastructure is deployed as expected and that it has all required features
  * Manual gate to confirm deployment to production

#### Stage - Production

* Production release process begins
  * Artefact deployed to production
  * Environment monitored for operational purposes
  * Handed over to data development team as development environment

### Test/Production Environment Infrastructure

This lifecycle is used to create the ARM templates which are used to deploy the test and production infrastructure to run the developed data pipelines. Note that this lifecycle does not include a production stage since the artefacts will be consumed from the main pipeline authoring process.

<table>
<tr>
<td width="15%">&nbsp;</td>
<td width="70%"><img src="images/lifecycleProd.png" alt="Flow of production infrastructure lifecycle as described in the text below" /></td>
<td width="15%">&nbsp;</td>
</tr>
</table>

#### Stage - Development

* ARM Templates will be written in Visual Studio/VSCode (or some other tool)
* Templates will be committed to the repository in a specific folder under the feature branch
* Manual deployment testing will be done with the ARM templates
* Pull request from feature branch to collaboration branch
* Approval of the pull request will trigger the build process
  * Build process copies ARM templates to create an artefact

#### Stage - Testing

* Artefact is deployed into a testing environment
  * Automated/manual tests confirm that infrastructure is deployed as expected and that it has all required features
  * Manual gate to confirm tagging of artefact for production use


### Tests and test data

This lifecycle is used to produce tests and test data which can be used against various aspects of the environment including the data pipelines. Optionally this lifecycle can be merged with the main pipeline development process. For cost reasons, however, the data generation should be deployed separately as there is usually not a good reason to regenerate test data on every deployment cycle.

When a new data pipeline is written, tests should also be created to ensure that it runs properly. These may be manual tests or automated tests, but they must be repeatable. An example of this might be that a pipeline creates aggregates values in a file from input data. In this instance we would create some known data with known dates and run the pipeline against that data. Since we know the data, we know the expected results. This test can check for issues with dates and times (leap years, summer time, time zones) as well as a general sanity check. Other tests should be written around the functionality of the pipeline and data created to run the tests against with known results. Data should be created to test all edge cases as well as known good cases. For instance, numerical fields should have rows with non numerical data, dates should have known bad values such as 02/03/1799 or dates formatted incorrectly such as 3rd March 2020 when the process expects 03/03/2020 and so on. Values should be created either side of midnight and during the leap day as well as during daylight savings transitions from multiple time zones if appropriate. Test triggers should be created to test specifically the date range of the test data, ideally with tumbling window style triggers since these are easy to test specific date ranges and designed for this purpose. Outputs from each ADF run should then be tested to ensure the correct number of rows were processed and the correct outputs created. A good example of a failure here would be during daylight savings transitions a transaction at 00:59 should be processed on the previous day in some circumstances and so row counts will be affected, potentially missing some data or counting it twice. 
In the case of value format errors it might be generally favourable to allow bad data through and use data quality processes in the pipeline to deal with it, rather than failing the pipeline. While it might seem obvious to fail the pipeline and try to fix everything, this is operationally problematic so remember that at the Data Factory level testing should be limited to whether the pipeline itself will produce known results. Data quality testing will be performed within the production pipeline and will be tested separately. A data factory pipeline test failure will mean failure of the build/release while a data quality test failure might not fail the release, but rather inform the backlog that there is more work to do on data quality. These decisions will be specific to your testing processes.

<table>
<tr>
<td width="15%">&nbsp;</td>
<td width="70%"><img src="images/lifecycleTesting.png" alt="Flow of testing and test data lifecycle as described in the text below" /></td>
<td width="15%">&nbsp;</td>
</tr>
</table>

#### Stage - Development

* Tests written in an appropriate framework in a tool such as Visual Studio/VSCode
  * Tests committed to repository under a specific folder in the feature branch
  * Unit testing performed against tests to ensure they work as planned
* Pull request to merge code into collaboration branch
  * build process builds and compiles tests and creates artefact to be used in pipeline development processes

* Test data created in Excel to match the tests created
* ARM template created to provide suitable infrastructure for test data (Blob store, SQL Server etc.)
* Data deployment script created to load data into the infrastructure after deployment
* Items above committed to repository under specific folder in feature branch
* Manual testing performed to verify test data will deploy
* pull request to merge from feature to collaboration branch
* Build process creates artefact with assets in

#### Stage - Testing

* ARM template deployed to create infrastructure
* Data deployment script pushes data into infrastructure
* Tests run to verify deployment of correct data
* Manual confirmation of stage completion

#### Stage - Production

* ARM template deployed to create infrastructure
* Data deployment script pushes data into infrastructure

### Data Ingest Pipelines

When developing the actual pipelines, the development ADF environment is used to create the code and "compile it" into an ARM template. This template will then deploy a new ADF environment to be used for testing against the test data created above. Finally, once tests are passed the environment is destroyed and a production version deployed which points to the real data sources rather than test data. Optionally, a pre-production environment can be created for the purposes of testing against production data and checking the outputs. In this case, simply create a temporary output storage account as a data sink.

<table>
<tr>
<td width="15%">&nbsp;</td>
<td width="70%"><img src="images/lifecyclePipelines.png" alt="Flow of Data ingest pipeline lifecycle as described in the text below" /></td>
<td width="15%">&nbsp;</td>
</tr>
</table>

#### Stage - Development

* Create a feature branch in the repo for use in ADF, linked to work items in the backlog.
* Work on the pipelines in the development ADF environment created above
* Use the debug feature in ADF to perform unit testing on pipeline features
* Commit (save) the code and create a pull request

#### Stage - Testing
#### Stage - Production