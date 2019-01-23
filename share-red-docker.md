# Purpose
This document helps me collate my thoughts and map progress while solo-developing. It's for personal use only, however I may use it for referencing in remote communication with collaborators.

# What and Why

We don't want to maintain a fork of node-red. We want to use node-red as a npm package and design incrementally workable process for the use cases listed below:
### SR-1
Anyone in share-research should be able to upgrade share-red instance to a newer release of node-red
### SR-2
Allow a developer to build custom nodes and include them in the runtime
### SR-3
Allow a team to iterate over a flow using node-red projects
### SR-4
Build a cloud based runtime (For both editing and running?... tbd)
### SR-5
Use custom settings.js for share-research

# How
There is an existing community using a dockerized version of node-red (although not up-to-date with the latest node-red version). See [node-red-docker](https://hub.docker.com/r/nodered/node-red-docker/).
We can customize this to address our needs.

Docker has the benefit of easy portability from local development to cloud. [AWS ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html) is designed to build, run, test, and deploy distributed applications using containers.
