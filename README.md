### Personal Project kick off (4/10/2018)

```
The database I will use for this project is Redis
```
[YouTube link](https://youtu.be/VhOpmTJKOp4)
### AWS setup
#### Task
```
Set up the five nodes on AWS EC2 instances, as node1, 2, 3, 4, 5

Partion node1 and node2 away from node3, node4 and node5
```
#### Implement
```
1. AWS network setup.
	* Allocate new Elastic IP
	* Create new security group named cmpe281 and open port 80, 443 and 6379
	* Create a new VPC named cmpe281, and assign the new Elastic IP to it
	* Create two Public subnets under the VPC cmpe281 and named them Public subnet1 and Public subnet2 seperately

2. AWS instances setup
	* Launch two instances in Public subnet1 and assign them Public IP in configuration for further connection
	* Launch three instances in Public subnet2, the configuration is the same as above

3. Deploy redis on five instances
	* Connect to AWS EC2 instance using ssh with key in the same folder
	* Create document file named "hosts.txt" to store all the servers' DNS information
	* Install pssh and using it to operate all five nodes at the same time

	brew install pssh

	* Update instance package and install redis

	pssh -i -h hosts.txt -l ec2-user 'wget http://download.redis.io/redis-stable.tar.gz'
	
	pssh -i -h hosts.txt -l ec2-user 'tar xvzf redis-stable.tar.gz'

	pssh -i -h hosts.txt -l ec2-user 'sudo yum -y update'

	pssh -i -h hosts.txt -l ec2-user 'sudo yum -y install gcc make'

	pssh -i -h hosts.txt -l ec2-user 'cd redis-stable; sudo make distclean'

	pssh -i -h hosts.txt -l ec2-user 'cd redis-stable; sudo make'

	pssh -i -h hosts.txt -l ec2-user 'cd redis-stable; sudo make install'

	* Modify master's redis config file 

	comment bind
	change protected-mode to 'no'

	* Modify slave's redis config file 

	comment bind 
	change protected-mode to 'no'
	uncomment slaveof to the master's IP address and port

	* Modify sentinel config file

	change sentinel monitor to master's IP address and port
	uncomment protected-mode to 'no'

	* Start server in following order using screen which allows server run in background

	Master -> Slave -> Sentinel

	screen 

	redis-server redis.conf

	ctrl a, ctrl d (detach screen)

	->

	screen 

	redis-sentinel sentinel.conf
```
#### Challenge
```
1. After instances launched in Public subnet2, instance in Public subnet1 can't connect to these instances
* Solution: Change route table in subnet2 to the same one with subnet1
2. Cannot connect to instance via pssh
* Solution: I log into each instance via ssh for one time, then using pssh to operate on 5 instances at the same time
3. During deployment, redis server can't work properly
* Solution: Comment out protected-mode to 'no' in redis config file
```


### AP properties test
#### Task
``` 
Test Availability when partion occurs, all the data in different nodes are still 
available for read.
```
#### Implement
```
	* All the redis-server and redis-sentinel are up and running

	* Set a new value in master server which can be access in all the slaves

	* Apply Network ACLs to Public subnet1, which will block port 6379 and 26379

	* Network partition successful, a new master is selected in Public subnet2

	* Add new value in old master, add new value in new master

	* Restore network

	* value in the old master is dismissed, value in the new master can be accessed in all the slaves
```
#### Challenge
```
1. After all the servers been properly set up, block port 6379 and 26379 on Public subnet1, sentinel server can't detect block and select new master server
* Solution: I check the security group of the VPC, everything seems alright. Then I check the Network ACL setup, I find out that the order of the network protocol will effect the result. I put block protocols on the top and allow all the other traffic at the bottom, restart the whole process. Everything works. 
```
