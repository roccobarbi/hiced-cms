# README

Hiced-CMS is intended to become an open-source, higly distributed and scalable CMS based on microservices, mostly written in Go.

## Table of Contents

- [The hiced-cms project](#the-hiced-cms-project)
- [Frontend components](#frontend-components)
- [Backend components](#backend-components)

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

## Frontend components

Frontend components are those components that are required to receive a user's request from the internet and deliver the page to her.

For the first version of hiced-cms, the process will require the following interactions:
1. a __request manager__ receives a request from the web server (such as nginx or apache), assigns it an id and posts it to the request queue;
1. a __language validator__ picks up the oldest request from the request queue, validates the language (using the path, the domain or some other signal such as a cookie value), adds this information and posts the request to the language queue;
1. a __path validator__ picks up the oldest request from the language queue, checks if the content id exists (issuing an error if needed) and validates the path. If the path is not canonical, a redirect response is issued. Then the requested is added to the path queue;
1. a __response builder__ picks up the oldest request from the language queue, if it's a redirect it immediately issues a response ready event, otherwise it sends requests to the template fetcher and the content fetcher. When they respond, it builds the response payload and it issues a _response ready_ event;
    - a __template fetcher__ receives a request for a content id, checks a cache for its template, otherwise it gets it from the DB, otherwise it updates the cache and it responds;
    - a __content_fetcher__ receives a request for a content id, checks a cache for any contents associated to it, otherwise it gets it from the DB, otherwise it updates the cache and it responds;
1. the __request manager__ (which has to subscribe to the _response ready_ event), receives the event, checks if the request is in its memory and, if so, it responds to the request and it forgets about it.

A __logger service__ also subscribes to the _response ready_ event. It formats a line of lig and posts it to the endpoint that has been set up for it (e.g. a lambda service to store logs on Athena).

## Backend components

Backend components are those components that are required to manage the website, performing tasks such as:
- creating templates;
- writing contents;
- managing comments.