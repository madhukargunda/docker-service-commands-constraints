# docker-service-constraints
How to control the placements of the containers in to different nodes using docker CLI.

Five different ways we can control the placememt of the docker containers in different nodes.


# Controlling Container Placement in Swarm

## Service Constraints

```
docker node ls

docker service create --name app1 --constraint node.role==worker nginx

docker service update --constraint-rm node.role==worker --constraint-add node.role=manager app1

docker node update --label-add dmz=true node2

docker service create --name dmz-nginx --constraint node.labels.dmz==true --replicas 2 nginx

```
## Service Mode

```
docker service create --mode=global --name test1 nginx

docker service ls

docker service rm test1

docker service create --mode=global --name test1 --constraint=node.role==worker nginx

```
## Service Placement Preference

```
docker node update --label-add azone=1 node1

docker node update --label-add azone=2 node2

docker node update --label-add azone=2 node3

docker service create --placement-pref=spread=node.labels.azone --replicas=2 --name webapp1 nginx

docker service create --placement-pref=spread=node.labels.azone --replicas=2 --name webapp2 nginx

docker service update --placement-pref-rm spread=node.labels.azone webapp1

docker service scale webapp2=8

docker service update --constraint-add node.role==worker webapp2
```

## Node Availability

Different status of the nodes

- active : By default when node created its status is active.

- pause  : When node went to pause status , Node runs the exisiting tasks but not allowed for new tasks.
           This helps us troubel shooting the issues.
           
- drain : When node went to drain status ,
          it will not allow for the new tasks and all exisiting tasks will be rescheduled to different nodes.

```
docker service create --name webapp1 --replicas 4 nginx

docker node update --availability=pause node2

docker service update --replicas=8 webapp1

docker node update --availability=active node2

docker node update --availability=drain node3

docker node ls

docker node update --availability=active node3
```
## Service Resource Requirements
```
docker service create --reserve-memory 800M --name 800 nginx

docker service create --replicas 4 --reserve-memory 300M --name 300 nginx

docker service update --reserve-memory 0 800

docker service create --reserve-cpu 8 --name 8 nginx

docker service ps 8

docker service create --limit-memory 100M --name 100 bretfisher/stress:256m

docker service ps 100
```

### Representing the constraints in docker stack file

```
account-service-db:
    user: "1000:50"
    image: mysql
    ports:
      - "3306:3306"
    command: --innodb_use_native_aio=0
    deploy:
     replicas: 1
     placement:
      constraints:
       - node.labels.mysql.node == true
     restart_policy:
      condition: on-failure
      delay: 10s
    networks: 
     - back-end

```
### Docker Build in LABELS

```
node.id
node.hostname
node.ip
node.role (manager | worker)
node.platform.os (linux | windows)
node.labels (Default empty)

```
### Docker Service Logs

```
docker service ls

docker service logs account_service

docker service ps account_service

docker service logs TASK ID

docker service logs --tail 5 -- follow account_service

docker service logs --tail 5 --raw --no-trunc account_service

docker service logs account_service 2>&1 | grep SEARCH TERM

docker service logs account_service2>&1 | findstr SEARCH TERM
```
