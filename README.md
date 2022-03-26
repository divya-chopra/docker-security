# docker-security

DOCKER DAEMON SECURITY: 
Running containers with Docker implies running the Docker daemon. Common ways to secure docker Daemon  

1. Only trusted users should be allowed to control your Docker daemon.  
2. If you run Docker on a server, it is recommended to run exclusively Docker on the server.  
3. It is essential to patch both Docker Engine and the underlying host operating system running Docker to avoid any vulnerabilities.   
4. Improve Container Isolation  
  4.1 Use Namespace  
  4.2 Use CGroups  
  4.3 Use Linux Capabilities  
5. Limit Container Resources   


1. NAMESPACES:  
  
-> It is a linux feature that is used by docker also.  
-> When we start container, docker automatically creates namespace for the container.  
-> It provides container isolation in terms of process, network, hostname etc.  
-> Processes running in 1 container cannot see any processes of another container or host system.   
-> Advantage:   
  1. If 1 container is breached, attacker will not be able to connect or see any other processes of other container or host system.   
  2. It provides impression of running a container on its own operating system. They are the basis of container isolation.  

Experiment:   
1. Check processes in host system:   
Command for linux: ps -ef  
Command for windows: tasklist  
2. Create a container, login into it and check the processes:   
Command: docker run -dt --name namespace01 busybox sh  
Command: docker exec -it namespace01 sh  
Command: ps -ef  
hostname  

The running process in container will be different from the running processes of host machine. This isolation is the result of namespace.  

2. CGROUPS:   

-> Also a Linux Kernel Feature.  
-> Isolates Resource usage(CPU, Memory, Disk I/O, Network)   
-> It ensures that each container gets its share of CPU/ Memory/ Disk  
-> Advantage:  
  1. If we have 2 containers in host machine. If a DDoS attack happens which is sending multiple request from multiple sources to one of the container. That container will start consuming huge amount of cpu/memory to serve all those requests. If we limit the amount of CPU/Memory so that when attack happens, it will not be able to take more CPU/ RAM then we assigned.   
  2. Single container will not bring down the whole system by exhausting resources.  

RESTRICTING RESOURCES:  

-> By Default, a container has no resource constraints. It can use as much as host allows.  
To check how much host allows:  
Command: docker info  
![image](https://user-images.githubusercontent.com/59343209/160227798-b92761fe-0e4d-40b5-bb85-baea245c79c1.png)  

-> Restrict Memory:  
To check all options available for memory (-m)  
Command: docker run --help  

![image](https://user-images.githubusercontent.com/59343209/160227830-0147bc56-24f0-4a6c-8bdb-19aaf0f23e14.png)  

--memory -> Hard Limit -> Maximum limit of memory that is assigned to a container   
--memory-reservation -> Soft Limit -> Also known as reservation. Container will try best to maintain this limit but it can go beyond that.   

Command: docker container run -dt --name memlimit01 -m 256m busybox sh  
Command: docker container exec -it memlimit01 sh  
  cd /sys/fs/cgroup/memory  
  cat memory.limit_in_bytes  
  
Command: docker container run -dt --name memlimit02 --memory 500m --memory-reservation 256m amazonlinux  
Command: docker container exec -it memlimit02 bash  
  cd /sys/fs/cgroup/memory  
  cat memory.limit_in_bytes -> To check limit   
  cat memory.soft_limit_in_bytes -> To check reservation  
  
  ![image](https://user-images.githubusercontent.com/59343209/160228623-294f933b-9816-4022-84ac-9d259d92f21d.png)  

  
-> Restrict CPU:   
To check all options available for cpu (-cpus)  
Command: docker run --help  

![image](https://user-images.githubusercontent.com/59343209/160228643-82b88deb-c2f3-4b13-8fd9-05c311e38b81.png)  

-c or --cpus : Number of cpus  

Command: docker container run -dt --name cpulimit --cpus=0.5 busybox sh  

3. CAPABILITIES:  

-> Linux feature  
-> To have granular access   
-> When you run a container, it has capabilities that can be assigned/ un assigned.   
-> Some Capabilities are allowed by default. And some capabilities that are not added by default but can be added.   
Link: https://docs.docker.com/engine/reference/run/  
-> Command to add or remove capabilities:  

docker container run -dt --name addcap --cap-add LINUX_IMMUTABLE amazonlinux  
docker container run -dt --name dropcap --cap-drop KILL amazonlinux  

Example:   

1. docker container run -dt --name defcap amazonlinux  
   docker container exec -it defcap bash  
   touch test.txt  
   yum whatprovides /usr/sbin/useradd  
   yum install -y shadow-utils  
   useradd testuser  
   chown testuser:root test.txt  
3. docker container run -dt --name dropchowncap --cap-drop CHOWN amazonlinux  
    docker container exec -it dropchowncap bash  
    touch test.txt  
    yum whatprovides /usr/sbin/useradd  
    yum install -y shadow-utils  
    useradd testuser   ---> Not permitted  
    chown testuser:root test.txt    ---> Not permitted  
    
Best Security Practice: Allow minimum capabilities that are must to run the application.   
