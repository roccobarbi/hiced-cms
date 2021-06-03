# README

Hiced-CMS is intended to become an open-source, higly distributed and scalable CMS based on microservices, mostly written in Go.

## Important disclaimer for the usage of this repository

This repository has been created only with the intent of tracking the "big picture" for the hiced-cms project. Therefore, it should only be used to track:
- what individual microservices should make up the project (linking them as submodules);
- how the individual microservices should interact;
- how the CMS should behave as a whole.

Issues and specs relative to individual microservices should only be tracked in the applicable service's repository.

## The hiced-cms project

The rationale behind hiced-cms is that different tasks that are commonly performed by CMS software, such as managing the URLs, retrieving templates, retrieving contents, etc. can have different computational costs and can be performed independently (and sometimes in parallel, such as the retrieval of templates and contents). Moreover, the same infrastructure that is needed to deliver whole page can sometimes be required for other purposes (e.g. a service that retrieves contents could also be used by an API that delivers new contents to a live page). Using a microservices-based architechture allows the website's owner to scale it more efficiently and to make the most efficient use of those resources that can be repurposed:
- different services can be scaled independently of each other as demand grows, depending on their computational cost;
- the same service can be used by the CMS and by some other service that was developed independently from hiced-cms;
- the CMS as a whole can also be expanded upon without any need for the project to make the customization canonical.

Hiced-cms will be designed with most use-cases in mind. At the extremes, it should be possible to use it both as a single-server CMS (with all services coexisting on different localhost ports) and as the infrastructure that serves a very-high-traffic website from a large cluster of servers in the cloud.

To cater to the very-high-traffic scenario, hiced should be designed in a way that makes it as easy as possible to leverage the most important cloud providers.

My initial focus will be on AWS, but contributors are more than welcome to add support for any other service, such as Google Cloud, Azure, or even smaller providers. The software should be able to read the information needed to set itself up correctly from a configuration file, ideally in a format that is both human-readable and machine-readable (the best candidate for now is yaml). From the point of view of AWS, as a bare minimum, it should be easy to:
- set up service-specific AMIs that scale up based on traffic;
- use SQS to pass messages between services, so as to leverage clusters of service-specific machines more efficiently;
- store media on S3 and link directly to it;
- use a lambda endpoint to recive, resize, store and link each media element correctly when a new content is loaded;
- leverage both AWS DB offering, or a locally/remotly installed DB;
- leverage cloudfront for URLs that are not expected to change too often.