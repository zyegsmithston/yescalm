# yescalm
Possessed of great energy, discernment, genius and understanding

# Starting Out
- Install Nomad
- Set up a server and clients/agents
- Create a basic microservice
- Route that fucker
- DB - colo or cloud

## Install Nomad
This is a pretty easy bit.  Background: Hashicorp Nathaniel Nomad is a highly available, distributed, data-center aware cluster and application scheduler.
The official blurb is:
```
Nomad is a flexible workload orchestrator that enables an organization to easily deploy and manage any containerized or legacy application using a single, unified workflow. Nomad can run a diverse workload of Docker, non-containerized, microservice, and batch applications.
```

It's a single binary less than 100mb.  Used for server or client.  Distributed and resilient.

You can run it across datacentres and betwixt cloud and ground.  Really scales.

- Installing it
Download the right binary from here: https://www.nomadproject.io/guides/install/index.html#precompiled-binaries

Can you do this:
```
$ nomad -v
Nomad v0.9.3 (c5e8b66c3789e4e7f9a83b4e188e9a937eea43ce)
```

Then it's installed.

## Set up Nomad Server and Agents
From the manual:
```
The Nomad agent is a long running process which runs on every machine that is part of the Nomad cluster. The behavior of the agent depends on if it is running in client or server mode. Clients are responsible for running tasks, while servers are responsible for managing the cluster.

Client mode agents are relatively simple. They make use of fingerprinting to determine the capabilities and resources of the host machine, as well as determining what drivers are available. Clients register with servers to provide the node information, heartbeat to provide liveness, and run any tasks assigned to them.

Servers take on the responsibility of being part of the consensus protocol and gossip protocol. The consensus protocol, powered by Raft, allows the servers to perform leader election and state replication. The gossip protocol allows for simple clustering of servers and multi-region federation. The higher burden on the server nodes means that usually they should be run on dedicated instances -- they are more resource intensive than a client node.

Client nodes make up the majority of the cluster, and are very lightweight as they interface with the server nodes and maintain very little state of their own. Each cluster has usually 3 or 5 server mode agents and potentially thousands of clients.```

Start with an agent:
`nomad agent -dev`

This'll give a long output starting with:
```
==> No configuration files loaded
==> Starting Nomad agent...
==> Nomad agent configuration:

       Advertise Addrs: HTTP: 127.0.0.1:4646; RPC: 127.0.0.1:4647; Serf: 127.0.0.1:4648
            Bind Addrs: HTTP: 127.0.0.1:4646; RPC: 127.0.0.1:4647; Serf: 127.0.0.1:4648
                Client: true
             Log Level: DEBUG
                Region: global (DC: dc1)
                Server: true
               Version: 0.9.3

==> Nomad agent started! Log data will stream in below:
```
Client - If this is running in client mode
Log level - Standard stuff really
Region - This is Nomad specific: configurations will run in multiple defined datacenters (your own slicing of your environments be they physical or virtual)  Default is global dc1
Server - True because we're doing an all-in-one with the agent and -dev flags combined.
Version - Need I explain

*NOTE* You can access the UI via:
`http://localhost:4646/ui`

On startup the agent fingerprints the host and register the drivers available (gpu, docker, java, etc) while also trying to contact other nomad servers and consul for backend/key value storage.

Servers must join clusters on first boot with the `server join` command or just configuring it properly.

Nomad servers can run with garbage level permissions.
Nomad agents though, they need root permissions.

## Deploy a pissant job
This is the bog standard example from Hashicorp.  Fun though and quick example of how jobs work so don't bug me about them after this or I'll know you did read squat.

`nomad job init`
Will make an example.nomad file.  Open this sucker up and examine it and change the name if you like but don't feel you have to.  It deploys a redis instance, that's all.

`nomad job plan example.nomad`
```
+ Job: "example"
+ Task Group: "cache" (1 create)
  + Task: "redis" (forces create)

Scheduler dry-run:
- All tasks successfully allocated.

Job Modify Index: 0
To submit the job with version verification run:

nomad job run -check-index 0 example.nomad

When running the job with the check-index flag, the job will only be run if the
server side version matches the job modify index returned. If the index has
changed, another user has modified the job and the plan's results are
potentially invalid.
```
If you get "Constraint: Missing drivers" then you might need to:
- restart your docker service
- run the nomad agent as root


