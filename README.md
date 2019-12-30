#### Docker Swarm

- A swarm consists of multiple Docker hosts which run in swarm mode and act as managers (to manage membership and delegation) and workers (which run swarm services). A given Docker host can be a manager, a worker, or perform both roles. When you create a service, you define its optimal state (number of replicas, network and storage resources available to it, ports the service exposes to the outside world, and more). 

#### Creating swarm cluster 
```
[root@ip-172-31-39-46 ~]# docker swarm init
Swarm initialized: current node (uwv58yrstnl2nv8k9g8wp6f3k) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0pdpqqtm8gf64f24h6mg1s8pspkwt92uybqcrrbwjkgc48tuig-eqk5slt82dhxsva6ile9kdnei 172.31.39.46:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

#### Docker Overlay Network
```
[root@ip-172-31-39-46 ~]# docker network create --driver overlay wpnet
jx4d66uzc0zui4aoujy3jj24m
[root@ip-172-31-39-46 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
4cff255f6dd8        bridge              bridge              local
56dc62949ae5        docker_gwbridge     bridge              local
1bc803c707e0        host                host                local
t0l13ghuzm84        ingress             overlay             swarm
9fd3a981ed3b        none                null                local
jx4d66uzc0zu        wpnet               overlay             swarm
[root@ip-172-31-39-46 ~]# docker service create --name redis --network wpnet --replicas 1 redis:latest
k688dg6a844b9yw2de1140ppj
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```
#### Creating redis service
```
[root@ip-172-31-39-46 ~]# docker service create --name redis --network wpnet --replicas 1 redis:latest
k688dg6a844b9yw2de1140ppj
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 


[root@ip-172-31-39-46 ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
k688dg6a844b        redis               replicated          1/1                 redis:latest        
```
#### Creating Flask service
```
[root@ip-172-31-39-46 ~]# docker service create --name flask --network wpnet -p 80:8080 --replicas 2 -e redis_host=redis -e redis_expiry=300 fujiclado/flask-redis:latest
s2g692oxz2wonyd2h0ddyhal5
overall progress: 2 out of 2 tasks 
1/2: running   [==================================================>] 
2/2: running   [==================================================>] 
verify: Service converged 

--


[root@ip-172-31-39-46 ~]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
s2g692oxz2wo        flask               replicated          2/2                 fujiclado/flask-redis:latest   *:80->8080/tcp
k688dg6a844b        redis               replicated          1/1                 redis:latest                   

[root@ip-172-31-39-46 ~]# docker service ps flask
ID                  NAME                IMAGE                          NODE                                          DESIRED STATE       CURRENT STATE                ERROR               PORTS
s9s7j5cp6k02        flask.1             fujiclado/flask-redis:latest   ip-172-31-39-46.ap-south-1.compute.internal   Running             Running about a minute ago                       
w7f6ipri2nef        flask.2             fujiclado/flask-redis:latest   ip-172-31-39-46.ap-south-1.compute.internal   Running             Running about a minute ago                       


[root@ip-172-31-39-46 ~]# docker service ps redis
ID                  NAME                IMAGE               NODE                                          DESIRED STATE       CURRENT STATE           ERROR               PORTS
nk4nph5zijwp        redis.1             redis:latest        ip-172-31-39-46.ap-south-1.compute.internal   Running             Running 4 minutes ago                 
```
#### Increasing replicas count
```
[root@ip-172-31-39-46 ~]# docker service update --replicas 4 flask
flask
overall progress: 4 out of 4 tasks 
1/4: running   [==================================================>] 
2/4: running   [==================================================>] 
3/4: running   [==================================================>] 
4/4: running   [==================================================>] 
verify: Service converged 



[root@ip-172-31-39-46 ~]# docker service ps flask
ID                  NAME                IMAGE                          NODE                                          DESIRED STATE       CURRENT STATE            ERROR               PORTS
s9s7j5cp6k02        flask.1             fujiclado/flask-redis:latest   ip-172-31-39-46.ap-south-1.compute.internal   Running             Running 3 minutes ago                        
w7f6ipri2nef        flask.2             fujiclado/flask-redis:latest   ip-172-31-39-46.ap-south-1.compute.internal   Running             Running 3 minutes ago                        
xnotkrd2y4e2        flask.3             fujiclado/flask-redis:latest   ip-172-31-39-46.ap-south-1.compute.internal   Running             Running 52 seconds ago                       
icj7io4i7ax3        flask.4             fujiclado/flask-redis:latest   ip-172-31-39-46.ap-south-1.compute.internal   Running             Running 52 seconds ago         
```
#### Listing swarm nodes 
```
[root@ip-172-31-39-46 ~]# docker node ls
ID                            HOSTNAME                                      STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
uwv58yrstnl2nv8k9g8wp6f3k *   ip-172-31-39-46.ap-south-1.compute.internal   Ready               Active              Leader              18.09.9-ce
```


