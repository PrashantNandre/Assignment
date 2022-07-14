* Assignment 1
Q. Write a common use-case, where you will use a daemon set instead of replica set.

Replica set is used to ensure that defined set of Pods are running all the time.
daemon set is used to ensure that all nodes are running the copy of pod.
daemon sets are commonly used for logging and monitoring for hosts. 
for example a node needs a service which collects the vaccination data and stores into database.
if you want to get data from each node, the best option is to schedule a container on every node that will gather these data from each individual node. 
if you schedule one container to gather data from all nodes, you would run the risk that the node on which the service is running dies for whatever reason, and you’d lose data from the whole cluster. 
but having a copy of the same container on every node will help.


*Assignment 2
Q. Suppose you have deployed your application using a deployment controller. Assume the initial number of replicas is one. Write the steps needed to update a container's image using deployment, making sure that there is zero downtime.

step1 : kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'
step2 : kubectl set image deployment kubia nodejs=luksa/kubia:v2

*Assignment 3
Q. You have deployed an application, that is listening at port 8000. You used a replica-set to deploy it and created a NodePort service to make it accessible. But when you test it, somehow the application is not reachable at the port. Write down your approach and sequentially all the steps that you will take to find out the issue.

check node port,IP,specs,labels,selector, in YAML file
check the syntax

*Assignment 4
Q. VOTING APP -DEMO practical

Commands : 
	git clone https://github.com/ashishrpandey/example-voting-app
	cd example-voting-app/
	cd k8s-specifications/
	kubectl apply -f . 
	kubectl get svc

1)delete vote pod :
	kubectl delete pod vote-94849dc97-w4ft6
	kubectl get pod

Observations : 
	new container get created
	still voting app is accessible
	no problem in no. of votes
	voting app didn't have anything to store so
	voting app, redis app, worker app, db app and result app worked fine
	for large application for continuity we can scale the deploymen

2) delete worker pod 
	kubectl delete pod worker-dd46d7584-2qm6j
	kubectl get pod
	kubectl logs worker-dd46d7584-k6chc

Observations : 
	still voting app is accessible
	no problem in no. of votes
	worker app able to pull the data
	system flow is not disturbed
	voting app, redis app, worker app, db app and result app worked fine
	while new worker app is coming logs got deleted	

3) delete db pod 	
	kubectl delete pod db-b54cd94f4-jkbxf
	kubectl get pod
	kubectl logs db-b54cd94f4-zrkj7

Observations : 
	impact on records, data got deleted
	new db pod able to connect and do the insertion of votes
	connection to result app is ended by db app
	socket is not established
	impact on result shown on screnn i.e result app

4) To Start the resultant app => the socket should be established
				i.e restart the result app
				kubectl delete pod result-5d57b59f4b-vs2lb

Observations : 
	now result app is working , showing the results


Kubernetes supports Stateless application
take care of data, Kubernetes doesn't support data

Voting App IP = http://13.215.250.93:31000/
Result IP = http://13.215.250.93:31001/

*snapshots of logs

[root@ip-172-31-21-48 k8s-specifications]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
db           ClusterIP   10.105.144.195   <none>        5432/TCP         2d11h
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          5d11h
kubia        ClusterIP   10.103.57.224    <none>        80/TCP           5d9h
kubia2       ClusterIP   10.99.10.99      <none>        80/TCP           5d10h
redis        ClusterIP   10.108.42.216    <none>        6379/TCP         2d11h
result       NodePort    10.111.216.130   <none>        5001:31001/TCP   2d11h
vote         NodePort    10.107.60.100    <none>        5000:31000/TCP   2d11h

[root@ip-172-31-21-48 k8s-specifications]# kubectl logs worker-dd46d7584-7rp24
Connected to db
Found redis at 10.105.31.205
Connecting to redis
Processing vote for 'a' by '463fdf08c105b5f'
Processing vote for 'a' by '463fdf08c105b5f'
Processing vote for 'b' by '463fdf08c105b5f'
Processing vote for 'a' by '463fdf08c105b5f'

[root@ip-172-31-21-48 k8s-specifications]# kubectl logs result-5d57b59f4b-vs2lb
Wed, 06 Jul 2022 16:28:27 GMT body-parser deprecated bodyParser: use individual json/urlencoded middlewares at server.js:67:9
Wed, 06 Jul 2022 16:28:27 GMT body-parser deprecated undefined extended: provide extended option at ../node_modules/body-parser/index.js:105:29
App running on port 80
Waiting for db
Connected to db
Error performing query: Error: This socket has been ended by the other party
