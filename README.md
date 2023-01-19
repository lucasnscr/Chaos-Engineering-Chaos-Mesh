[Chaos Engineering](https://principlesofchaos.org/) is the discipline of experimenting on a system in order to build confidence in the system’s capability to withstand turbulent conditions in production.

Advances in large-scale, **distributed Software** systems are changing everything related to Software Engineering. Flexibility of development and velocity of deployment are good examples, which show the change principles.

The good point about chaos process is: How much confidence we can have in the complex systems that are put in a production environment.

Resilience patterns improve any mechanism for distributed systems to be reliable, create scenarios of errors, identify weakness and aberrant behaviors. Systemic weakness could take the form of:

- Improper [fallback](https://github.com/lucasnscr/Resilience-Patterns/tree/main/circuitbreaker) settings when service is unavailable

- [Retry](https://github.com/lucasnscr/Resilience-Patterns/tree/main/retry) storms from improperly tuned timeouts

- Outages when a downstream dependency receives too much [traffic](https://github.com/lucasnscr/Resilience-Patterns/tree/main/rate-limiter)

- [Cascading failures](https://github.com/lucasnscr/Resilience-Patterns/tree/main/bulkhead) when a single point of failure crashes

For each error we have resilience patterns implementation in a [github repository](https://github.com/lucasnscr/Resilience-Patterns) and using **Java**, **Spring** and **Resilience4J**.


We need a way to manage the chaos inherent in these systems, take advantage of increasing flexibility and velocity and have confidence in production deployments.

#Chaos Engineering with Chaos Mesh
[Chaos Mesh](https://chaos-mesh.org/docs/) is an open source cloud-native Chaos Engineering platform. It offers various types of fault simulation and has an enormous capability to orchestrate fault scenarios. Using Chaos Mesh, you can conveniently simulate various abnormalities that might occur in reality during the development, testing, and production environments and find potential problems in the system. To lower the threshold for a Chaos Engineering project, Chaos Mesh provides you with a perfect visualization operation. You can easily design your Chaos scenarios on the Web UI and monitor the status of Chaos experiments.

## Chaos Mesh Architecture

Chaos Mesh is built on Kubernetes CRD (Custom Resource Definition). To manage different Chaos experiments, Chaos Mesh defines multiple CRD types based on different fault types and implements separate Controllers for different CRD objects. Chaos Mesh primarily contains three components:

- **Chaos Dashboard:** The visualization component of Chaos Mesh. Chaos Dashboard offers a set of user-friendly web interfaces through which users can manipulate and observe Chaos experiments. At the same time, Chaos Dashboard also provides an RBAC permission management mechanism.


- **Chaos Controller Manager:** The core logical component of Chaos Mesh. The Chaos Controller Manager is primarily responsible for the scheduling and management of Chaos experiments. This component contains several CRD Controllers, such as Workflow Controller, Scheduler Controller, and Controllers of various fault types.


- **Chaos Daemon:** The main executive component. Chaos Daemon runs in the DaemonSet mode and has the Privileged permission by default (which can be disabled). This component mainly interferes with specific network devices, file systems, and kernels by hacking into the target Pod Namespace.


![Chaos Mesh Architecture](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hwbkesk554n48fiai72q.png)

### Chaos Mesh Faults

**1. Basic resource faults**
  
  1.1. PodChaos: simulates Pod failures, such as Pod node 
       restart, Pod's persistent unavailability, and certain 
       container failures in a specific Pod.
  
  1.2. NetworkChaos: simulates network failures, such as 
       network latency, packet loss, packet disorder, and 
       network partitions.

  1.3. DNSChaos: simulates DNS failures, such as the parsing 
       failure of DNS domain name and the wrong IP address 
       returned.
  
  1.4. HTTPChaos: simulates HTTP communication failures, such 
       as HTTP communication latency.

  1.5. StressChaos: simulates CPU race or memory race.
  
  1.6. IOChaos: simulates the I/O failure of an application 
       file, such as I/O delays, read and write failures.

  1.7. TimeChaos: simulates the time jump exception.

  1.8. KernelChaos: simulates kernel failures, such as an 
       exception of the application memory allocation.

**2. Platform faults:**
  
  2.1. AWSChaos: simulates AWS platform failures, such as the 
       AWS node restart.

  2.2. GCPChaos: simulates GCP platform failures, such as the 
       GCP node restart.

**3. Application faults:**
  
  3.1. JVMChaos: simulates JVM application failures, such as 
       the function call delay. 

### Installing Chaos Mesh in Kubernetes Cluster

Beginning implementation of chaos mesh on kubernetes we used this **[Documentation](https://dev.to/lucasnscr/deployment-spring-application-in-kubernetes-with-minikube-4gl7)** or [clone this repository](https://github.com/lucasnscr/Kubernetes-Spring/tree/chaos-mesh) on branch chaos-mesh

Now with this repository, we need to have installed in your machine [docker](https://www.docker.com/) and [minikube](https://minikube.sigs.k8s.io/docs/start/). After this process you run this command to start the Kubernetes cluster.

```
minikube start
```

Now Installing Chaos Mesh:

```
#install chaos mesh
curl -sSL https://mirrors.chaos-mesh.org/v2.1.5/install.sh | bash

# check pods
kubectl get po -n chaos-testing

>> output
NAME                                       READY   STATUS    RESTARTS      AGE
chaos-controller-manager-bc4b9bf7b-2jd8m   1/1     Running   2 (15m ago)   7d3h
chaos-controller-manager-bc4b9bf7b-8qrnq   1/1     Running   2 (15m ago)   7d3h
chaos-controller-manager-bc4b9bf7b-pcxzc   1/1     Running   1 (15m ago)   7d3h
chaos-daemon-qmk4v                         1/1     Running   1 (15m ago)   7d3h
chaos-dashboard-599fc7bdf4-hf2sc           1/1     Running   2 (15m ago)   7d3h

# check services
kubectl get svc -n chaos-testing

>> output 
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                 AGE
chaos-daemon                    ClusterIP   None            <none>        31767/TCP,31766/TCP                     7d3h
chaos-dashboard                 NodePort    10.103.107.62   <none>        2333:30176/TCP,2334:31739/TCP           7d3h
chaos-mesh-controller-manager   ClusterIP   10.98.129.185   <none>        443/TCP,10081/TCP,10082/TCP,10080/TCP   7d3h

# access chaos mesh dashboard via NodePort service
# dashboard run on port 2333 as NodePort service
# dashboard can access via NodePort port 2333 mapping port, in my case port 31725
http://<minikube ip>:30176
http://192.168.49.2:30176/dashboard

# access dashboard via port forward 8888 -> 2333
# now the dashboard can be accessed via localhost:8888
kubectl port-forward -n chaos-testing chaos-dashboard-599fc7bdf4-hf2sc 8888:2333

http://localhost:8888
```

To finish the whole process, you access[Chaos Mesh Dashboard](http://localhost:8888)

![Chaos Mesh Dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e7ct6ep729xlh05uiqwz.png)

### Chaos Mesh Scenarios

There are two ways to run Chaos Mesh experiments. One method is directly creating experiments via the Chaos Mesh web dashboard. Another method is .yaml file deployments. If you need more details access [Chaos Mesh Documentation](https://chaos-mesh.org/docs/)


Api User Deployment.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: chaos

---

apiVersion: v1
kind: Service
metadata:
  name: user-api
  labels:
    name: user-api
  namespace: chaos
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  type: NodePort
  selector:
    app: user-api

---

apiVersion: v1
kind: ReplicationController
metadata:
  name: user-api
  namespace: chaos
spec:
  replicas: 3
  selector:
    app: user-api
  template:
    metadata:
      name: user-api
      labels:
        app: user-api
    spec:
      containers:
        - name: user-api
          image: lucasnscr/user-api:0.0.1
          ports:
            - containerPort: 8080
```

Now, we have the deployment and service created. Go create chaos for Api User, we are going create scenario pod failure:

```
kind: PodChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  name: chaos-failure
  namespace: chaos
spec:
  selector:
    namespaces:
      - chaos
    labelSelectors:
      app: user-api
  mode: one
  action: pod-failure
  duration: 30s
  gracePeriod: 0
```

Create another scenario of chaos, we now pod kill:

```
kind: PodChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  name: chaos-kill
  namespace: chaos
spec:
  selector:
    namespaces:
      - chaos
    labelSelectors:
      app: user-api
  mode: one
  action: pod-kill
  duration: 30s
  gracePeriod: 0
```

Now we created a stress test for Api User pod:

```
kind: StressChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  name: chaos-stress
  namespace: chaos
spec:
  selector:
    namespaces:
      - chaos
    labelSelectors:
      app: user-api
  mode: one
  containerNames:
    - ''
  stressors:
    memory:
      workers: 4
      size: '256MB'
  duration: 60s
```


![Chaos Scenarios Tests](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zdtuntu8k3kaxlu4l19g.png)

Chaos Mesh improves any situation, is a good tool for the Kubernetes environment and creates specific situations of chaos. 

Chaos Mesh is a project adopted by the [Cloud Native Computing Foundation](https://landscape.cncf.io/).

