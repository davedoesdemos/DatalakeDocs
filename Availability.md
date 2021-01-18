# Availability and SLAs

**produced by Dave Lusty**

## Introduction

This document aims to explain to some extent the availability of a solution and the risk of an outage to your service. This is an often misunderstood area and worth paying attention to in order to avoid surprises when things don't go your way.

## Compensation

It's important to get this out of the way right off the bat, if you're using generic off the shelf components you are very unlikely to be compensated for business losses. Modern cloud providers make it very clear what you can expect from the service and how to make your solution resilient to failures. As such it is up to you to ensure you don't incur losses as a result of an outage of a component or service. This will add cost to running your solution as you will need redundancy.
Compensation against failure to meet an SLA on a cloud platform is now almost universally a refund for down time of the affected service(s). This effectively means that if the service is not available you don't pay for it, just like if you'd switched it off.
Services where a provider is running a solution on your behalf, often one they designed and built, might offer financial compensation for your losses, but these will be individually negotiated as part of the contract, and will certainly add to the cost of that contract.

## The Service Level Agreement

Every service should include some kind of documented SLA telling you what kind of service availability to expect. You should read this document carefully and without any assumptions about what it covers. Lawyers write these documents and if they intend to say something they will definitely put those words in. Equally, if they don't include wording then they don't intend to provide that thing.
Generically, a service level agreement covers the service that the provider *intends* to provide. I also refer to this as a **Planned Uptime SLA** because it's more descriptive. The provider will design the solution such that maintenance activities will cause a certain level of unavailability per year. This is very definitely a cost/benefit decision, and the same provider might offer different SLAs against what look like identical services. Factors in this include:

 * Data Centre availability
 * Redundant Power (dual feeds)
 * Resilient Power (Generators and UPS)
 * Redundant WAN (multiple lines)
 * Resilient networking (redundant switching, routing, cabling)
 * Resilient storage (multipath IO, RAID, Erasure Coding)
 * Resilient hardware
 * Advanced low maintenance software (no reboots on update)

This is not an exhaustive list but shows that there are numerous factors in making an SLA. If a provider is able to update the network card firmware on a virtual host without stopping virtual machines on that host then a better SLA can be provided. If that virtual machine must be restarted in order to apply an update to the virtual hardware or drivers, then that reboot time must be subtracted from the SLA. If the restart is expected to take 5 minutes, and is likely to occur once per quarter, then 20 minutes will be subtracted from the SLA. Work through every component in the solution and you will arrive at a total expected downtime per year *on average*. This is then turned into a percentage and supplied as an SLA. An important factor to note is that most SLAs will include a provision that any specifically scheduled outages which you have been given a certain notice period for will be excluded from the SLA availability. For instance there might be a planned outage in a regional data centre, during which you are expected to move your workload elsewhere.

### What is availability

Availability is extremely difficult to define since it means different things to different people at different times. It is possible for something to be wrong with the service while it still carries out its main functionality. For instance, if an authentication service is unavailable, a web server might continue to provide web services to the public even if it is not possible for the owner to log in and make changes. In this example, most SLAs would consider the authentication service unavailable, but the web server available since it would work fine if you had a different way to authenticate.

### When is the outage

Knowing when there is an outage can be a huge difference in SLAs. Looking at a very simple business system which must be always available between 8am and 6pm we have to potential scenarios. In the first scenario we can use a server with a 99% SLA which will include maintenance whenever necessary (this used to be common, less so now). It is clear we cannot meet our business need with a server which has over 7 hours of downtime per month, so we cluster two machines together to improve availability. If we use update zones then we can assume the 7 hours of machine one will not overlap with machine 2, based on the documentation. In the second scenario we still have a server with a 99% SLA but this time we are able to choose the downtime window for updates. We may no longer require a second server since we can assume that the hours in which we require the server to be available it will be.

### Unplanned Outages

As you may have realised by now, *unplanned* outages are not part of the SLA. How could they be, they are unplanned by nature and therefore unexpected. While providers take enormous care to prevent unplanned outages, they do not make any statement about the likelihood of one happening. For this reason, you need to understand how the service may or may not fail and build your solution to take this into account. If the service itself offers multi-region failover then you may not need to plan for this. If the service doesn't offer a second copy of your data you definitely should plan for this.

### Data loss vs availability

An SLA which states that a storage service will be available 99.9% of the time states that you can use the service 99.9% of the time. You must also look for durability guarantees around data loss for the whole picture. It is entirely feasible that a storage service outage could result in data loss, so while you can write to and read from that service your data may no longer be stored on the service. For this reason it's important to have a copy of your data somewhere that you can recover from, or an alternative way to recover. If you have a virtual machine, you might not require a copy of the whole disk, but rather a way to redeploy that virtual machine to the same state. The former could include 100GB of data while the latter could be a 2KB script or template.

Planning only for availability of services could very easily lead to long outages for your solution.

### Composite or Compound SLAs

Combining SLAs to make a "solution" SLA is often discussed during architectural workshops. These are usually calculated by multiplying together the SLAs of individual components to produce an overall SLA. While seemingly useful, this is anything but and offers very little from an architecture or planning perspective.

An SLA is an agreement between the provider and the customer in which the customer agrees to pay a certain fee and the provider agrees to supply a service. There is no mechanism by which a provider will acknowledge the combining of these SLAs. If your storage service is unavailable you will not get a refund on your virtual machine service which relies upon it.

It is extremely unlikely that the maintenance of two services will be interrelated in a way not already considered in the individual SLAs, therefore for service planning you must use the individual SLAs.

Failure scenarios are not a part of the SLA, and so any consideration of failures is separate to the subject of SLA and therefore not related to that number.

The two services are probably not related and so multiplication of the SLA numbers is mathematically inappropriate since they do not represent the likelihood of a service being unavailable (risk, discussed below). The number produced by doing so does not relate to anything in either service and certainly not to both and is therefore effectively a random figure. If you added the expected annual outages from each service you would simply end up with a number representing the maximum expected maintenance downtime per year. That's only if all services had their allotted outage at different times which is unlikely in highly automated environments with strong SLAs. There is no architectural direction to be gleaned from this information, since each outage still must be addressed individually in the architecture to improve your outcome (i.e. less down time of your solution).

Going back to the example from above, a web service might remain available, making your solution fully available, even if the internal authentication mechanism is offline. Artificially connecting their SLAs in this scenario would be unhelpful and would likely lead to unnecessary changes to architecture. Instead, services should be considered individually and mitigating changes to architecture made where appropriate based on impact rather than arbitrary numbers.

## Risk

If you are interested in the risk of your solution becoming unavailable in various scenarios then that's a different subject entirely from SLAs and should be treated as such. This area is more mathematical and theoretical and so should not be considered part of a technical architecture, but rather as part of planning for risk management and costs.

### Mean Time Between Failure

MTBF is an often used term for systems. This is the average time to failure for a component of a certain type. For instance, mechanical hard drives will have a MTBF measured in hours. Using this number we know how often we can expect a hard drive to fail on average and therefore use that figure to work out how many drives to use in a RAID or Erasure Coding solution, as well as how many hot spares to keep available to keep the risks acceptably low. Alternatively, we could keep a second copy of data on some other medium, such as tape. While we know that eventually the single hard drive will fail, the likelihood of the disk and tape both failing catastrophically at the same time is acceptably low. In an architecture process the deciding factor would be time to recovery vs. cost, but both solutions reduce the risk of data loss. 

Knowing the MTBF of various components in a system, and the configuration of that system will give an expected failure rate for that device, which can be expressed as the risk of that item failing in a given time period. Adding a second identical system will reduce that risk accordingly. 

### Other factors

Outside factors can be brought in to your risk discussion. For instance the likelihood of disaster from weather events in a region. The South Eastern United States often suffers with hurricanes, while Iceland has volcanoes and South Western US is more likely to suffer an earthquake. Knowing this is the case we can mitigate those disasters by adding a second region to the solution if the risk is deemed unacceptable. As stated elsewhere in this document, if the risk is low then the second region might not need to be equivalent it just needs to be resilient enough to cope with exceptional circumstances. If the risk of earthquake is likely to cause 1 day of outage every 10 years, then the second region may only need to be designed to be unlikely to fail for one day.

### Solution Availability

Planning for your solution's availability may be a multi-faceted task. To properly plan you need to understand what must be available and in what capacity. It might not always be necessary to make the whole full featured service available. As ever this is a cost/benefit analysis and not a technical consideration.

Example: A retail web solution consists of clustered web and application servers with resilient storage and load balancers. All of this is configured in one region since the risk of regional failure is remote. The availability of this solution is very high, and could even be "five nines" because no maintenance action would impact the solution. It might be that there is a disaster recovery solution in place which requires several hours to implement because the cost/benefit of a live failover was not considered worthwhile. In this scenario we have an extremely unlikely scenario where the whole site might be offline for several hours. During that time, a static HTML page stored in Blob storage could be served to customers apologising for the outage. In this scenario, we have reduced the risk to the business by ensuring that *some solution* is always available while not necessarily incurring the expense of full and immediate recovery. The SLA of the static page could be very poor, but because it's a completely isolated system using different components in a different location, the overall risk can be considered very low from a business perspective.