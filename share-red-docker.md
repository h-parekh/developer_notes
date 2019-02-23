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
### SR-6
Queueing engine to send jobs to a node-red worker.
  1. Queue should send: URL to github flowfile
  2. Queue should be scalable
### SR-7
  1. Programmatically clone a git repo which contains a flow
  1. Programmatically install flow dependencies

# How
There is an existing community using a dockerized version of node-red (although not up-to-date with the latest node-red version). See [node-red-docker](https://hub.docker.com/r/nodered/node-red-docker/).
We can customize this to address our needs.

Docker has the benefit of easy portability from local development to cloud. [AWS ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html) is designed to build, run, test, and deploy distributed applications using containers.


# Progress

### 2019-02-13
* Added ability to clone flow from github project into NR runtime (See [commit](https://github.com/h-parekh/share-red-docker/commit/e7f9c4b5255384b4e86e21cec7215a96736f8d42)). Addresses [SR-7.1](#sr-7)
* I've researched about horizontal and vertical scaling of Bull Queue (Add whiteboard pic)
* Use [Queue#process](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueprocess) has a `concurrency` parameter
* Need to use a visual UI to test and play around with this. Couple options from Bull v3 docs:
  - [Taskforce](https://taskforce.sh/) - Paid version
  - [Arena](https://github.com/mixmaxhq/arena) - FOSS

### 2019-02-05
* Created a simple [queueing engine](https://github.com/h-parekh/share-queue-engine) with bull and redis. Addresses [SR-6.1](#sr-6)

### 2019-01-23
* Ryan added the custom node [share-lower-case](https://www.npmjs.com/package/share-lower-case) to npm, and I was able to add it as a dependency.

### 2019-01-22
* Created new repo [share-red-docker](https://github.com/h-parekh/share-red-docker)
* addresses the full requirement of node-red version upgrade in [SR-1](#sr-1)
* partially addresses [SR-2](#sr-2) by being able to handle custom nodes as dependencies. Needs more testing with 'share-lower-case'
* It also addresses [SR-5](#sr-5).
* Closes https://github.com/share-research/share-red/issues/11

# Running to do:
  1. Research spike: Articulate a strategy to maintain docker images, so we don't have to rebuild everytime
      - Create a new docker hub account owned by share-research?
  1. Research spike: Research data [mounting strategy](https://docs.docker.com/storage/) in Docker. Helps determine workflow for [SR-3](#sr-3)
    * Concern: Latency and conflict issues may occur with multiple node-red instances trying to access the same disk.
    * Filesystem conflicts during runtime? What files are written to the user directory when node-red executes a flow?
  1. ToDo: Clone github repo (Assumed and manually added for now)
    * There's two approaches:
      1. Clone the repository with git commands within the dockerfile at the time the image is built and update the `.config.json` of node-red runtime with the
      a json object like shown below. This is literally hacking into the runtime files and may cause unknown behavior. This is also the reason why loading an external volume with the git workflow doesn't automatically register it as a `NR project`
      ```json
      {
        "name": "share-red-flows-yet-another-name",
        "credentialSecret": "",
        "git": {
          "remotes": {
            "origin": {
              "url": "https://github.com/share-research/share-red-flows.git",
              "username": "",
              "password": ""
            }
          }
        }
      }
      ```
      2. Call a node-red API endpoint. There isn't an officially published API for this yet. Quick conversation with NR core devs informed me that this could
      be a feature in the future, and its unlikely to change fundamentally from the way it is right now. So I watched the http traffic from the browser while cloning a project and identified a POST /projects API that can do this operation.
  1. ToDo: Flow specific dependencies (Assumed and manually added for now)
    * This isn't a huge concern for me at the moment. I think its best handled to preload the docker image with all the required nodes to minimize initialization time.
  1. Sending message from Bull queue to spawn and run a flow
    - Start with one node-red instance per machine (concurrency: 1)
