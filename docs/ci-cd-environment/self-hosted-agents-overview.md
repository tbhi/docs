---
Description: This guide gives a brief overview of self-hosted agents and how they work.
---

# Self-hosted agents overview
Semaphore allows you to run your jobs in an environment which is controlled by your team. This is achieved through the use of self-hosted agents. Additionally, compared to a hosted platform, self-hosted agents offer more control over hardware, operating system versions, and the available software. This is because as you can run the agents anywhere you want: physical or virtual machines, containers, or in the cloud.

**Note:** Self-hosted agents are only available on our [enterprise plan](https://semaphoreci.com/pricing).

## Agent communication with Semaphore

All communication between the agent and Semaphore is one-way -- from the agent to Semaphore -- and is secured via HTTPS TLS 1.3.

When booting, the agent attempts to register with the Semaphore API. If it succeeds, it enters sync mode, sending periodic requests to Semaphore's API to tell it what it is doing and asking for instructions regarding what to do next. If it fails to register, the agent does not start and does not receive any jobs. If it fails to sync, it will not receive any further jobs and will eventually shutdown.

## Tokens used for communication

Three different types of tokens are used by the agent to communicate with Semaphore:

- **Agent type registration token**: this is the token that is used to start up the agent. It has only one purpose: register the agent on Semaphore. After registration, every agent gets a unique access token for further communication.
- **Agent access token**: this is the token used to coordinate the agent's state with Semaphore. It allows the agent to execute `POST /sync` and `GET /job` requests. When an agent fetches a job from Semaphore, it uses the log stream token to stream logs back to Semaphore.
- **Job logs stream token**: each job gets a unique log stream token to send back logs to Semaphore. This token is generated for each job and its lifetime is limited to the job's execution lifecycle.

## Available toolbox features

The Semaphore Toolbox offers a set of tools to navigate language versions, databases, cache, artifacts, and to check your code. Some of these features are tied to our hosted system, and are not available on self-hosted agents.

| Feature                                     | Available | Notes                                           |
|---------------------------------------------|-----------|-------------------------------------------------|
| Accessing the cache                         | Yes       | Using [S3 as a storage backend][cache with s3]. |
| Artifact storage                            | Yes       |                                                 |
| Publishing test results                     | Yes       |                                                 |
| Checking code with checkout command         | Yes       |                                                 |
| Changing language versions with sem-version | No        |                                                 |
| Managing databases with sem-service         | No        |                                                 |

Debug sessions and the ability to attach to running jobs is also not available, as Semaphore has no access to machines that are running on your infrastructure.

[cache with s3]: ./set-up-caching-on-aws-s3.md
