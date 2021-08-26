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

## Lifecycle

### Development

### Testing

### Production