Chapter 7 
Ephemeral volume vs. Persistent Volume
Volume types,
emptyDir: Empty directory in Pod with r/w access
hostPath: File or directory from the host node filesystem (only for single node clusters)
configMap, secret: Inject config data
nfs: An existing NFS share
persistentVolumeClaim: Claims persistent volume
For volume, first declare spec.volumes[] and then spec.containers[].volumeMounts[] inside pod definition.
PV is provided either by admin or assigned dynamically by mapping to a storage class.
In the pod, use the type PersistentVolumeClaim to mount abstracted PV by using PVC
Static approach: Create a storage device => reference it explicitly by PV.
Dynamic approach: No need for a PV object. Auto create from PVC setting spec.storageClassName.
kubectl does not allow the creation of PV. Only by manifest-first approach. spec.capacity and spec.accessModes must be defined. 
Attribute jostPath mounts directory from hostNode’s filesystem.
The reclaim policy determines what should happen with PV after it has been released from its claim.
Config options for PV: volume mode, access mode, reclaim policy
spec.volumeMode: filesystem or block mode. To see volume mode of existing volume, use -o wide option
spec.accessModes for how PV can be accessed:
RWO: R/W access by a single node.
ROX: Read only access by many nodes.
RWX: R/W access by many nodes.
RWOP: R/W access by a single node.
If dynamic, .reclaimPolicy attribute in the storage class
if static, spec.persistentVolumeClaim.
Retain (default): Released and can be reclaimed.
Delete: Remove PV and its associated storage.
Recycle: Deprecated
Static provisioning should use an empty string for the attribute spec.storageClassName if you do not want it to automatically assign the default storage class.
Using the describe command is a good way to debug if PVC was mounted.
After giving volume, spec.volumes[].persistentVolumeClaim(.claimName) is defined in Pod.
The storage class is used to provision a PV dynamically based on its criteria. No PV is written. Provisioner assigned to the storage class takes care of it.
Storage class is created only with YAML files (kubectl get storageclass)
In PVC, use spec.storageClassName attribute for dynamic provisioning.
If PVC is not created, the error is not thrown. Take attention.
Chapter 8
Init container vs side-car container
For init containers, a special attribute in pod spec.initContainers
For init containers, some attributes are like regular containers except probes cannot be used.
kubectl logs business-app -C configurer (for logs of a container)
side-car container is for synchronization, logging and watcher.
Formatted side-car after K8s 1.29 version.
The adapter pattern transforms the output produced by the application to make it consumable in the format needed by another part of the system.
The ambassador pattern provides a proxy for communicating with external services.
Chapter 9 - Labels and Annotations
Labels are an essential tool for querying, filtering and sorting K8s objects.
Annotations only represent descriptive meta-data for K8s objects but can’t be used for queries.
kubectl run labeled pod --image=nginx:1.25.1 --labels=tier=backend,env=dev
metadata.labels
To list the labels for all object types, or a specific object type: --show-labels
label commands:
To create label: kubectl pod some-pod region=eu
To edit: --overwrite ekle sonuna
To delete (minus sign) = kubectl label some-pod region - 
A label selector uses a set of criteria to query for K8s objects (available both in kubectl and manifest)
In command line (--selector or -l)
=, ==, !=, AND, OR, in, notion, exists can be used in queries
Deployments, services and network policies select a set of pods by labels.
The way of defining label selection can change according to types of objects and API version
K8s proposes a list of recommended labels which start with key prefix app.kubernetes.io.
Annotation examples: SCM commit hash IDs, release info, etc.
kubectl does not provide a command line option for annotations
only metadata.annotations
kubectl annotate similar to kubectl label
K8s and its extensions use annotations to configure run-time behavior for an object. pod-security.kubernetes.io/enforce:“baseline” is an example.
Chapter 10 - Deployments
A ReplicaSet is a K8s API resource that controls multiple, identical instances of a Pod running the application, so called replicas.
A Deployment abstracts the functionality of ReplicaSet and manages it internally.
The Deployment keeps the history of application versions and can roll back to an older version.
kubectl create deployment needs name and image name. Optionally --replicas. (spec.replicas)
spec.selector.matchLabels and spec.template.metadata.labels must match in terms of labels.
kubectl get deployments info:
READY : # of replicas available to end-users
UP-TO-DATE: # of replicas that have been updated to achieve the desired state.
AVAILABLE: # of replicas available to end users.
kubectl delete deployments …
Once a user changes the def of the Pod template in a Deployment, it will create a new ReplicaSet that applies changes to the replicas it controls and shut down the prev. ReplicaSet.
kubectl set image deployment …
kubectl edit deployment …
kubectl replace -f some.yaml (assumes image change)
kubectl patch .. (json gibi değişiklik)
kubectl rollout status deployment app-cache
kubectl history deployment app-cache (--revision 2 for more detailed view on a specific history point) 
Every change is represented by a so called “revision”
To change the # of revisions, use spec.revision. HistoryLimit (default to it)
To change default update strategy: spec.strategy.type:
To add cause: kubectl annotate deployment app-cache kubernetes.io/change-cause=”some reason.”
kubectl rollout undo deployment app-cache (--to-revision=1)
To observe creation of replicas in real time: k get pods -w
The Horizontal Pod AutoScaler (HPA) is an API primitive that defines rules for automatically scaling the # of replicas under certain conditions.
HPA uses metrics collected by metrics server (a target value, and average value/average utilization of a specific metric).
kubectl autoscale deployment app-cache --cpu-percent=80 --min=3 --max=5 (due to average CPU utilization, min/max # of pods)
To make HPA work properly, one should define resource requirements in a pod template.
Chapter 11 - Deployment Strategies
Rolling Deployment Strategy: old version to the new in batches
spec.strategy.type: Rollout
	spec.strategy.RollingUpdate.maxUnavailable
	spec.strategy.RollingUpdate.maxSurge 
Last two can be a fixed number or percentage.
maxUnavailable specifies the max # of Pods that can be unavailable during the update process.
maxSurge specifies the max # of Pods that can be created over the desired # of Pods. 
Define the readiness probe for the Pod template to ensure that a replica is ready to handle incoming requests.
spec.minReadySeconds: seconds before being available to incoming requests.

Fixed Deployment Strategy: will terminate replicas with the old version at once before creating another ReplicaSet that controls replicas running the new app version.
spec.strategy.type: Recreate

Blue-green Deployment Strategy: Both application versions will be operated at the same time with an equal # of replicas.
K8s routes traffic to the blue deployment, while the green deployment is rolled out and tested by the development or the test team.
Traffic will be switched over to the green deployment as soon as it is considered prod-ready.
Not a built-in strategy.
Create two deployments with selector labels: blue and green => Run both => Change the service label selector type from blue to green.

Canary Deployment is similar to blue-green. But, in this one, green deployments are only available to a subset of users (ex. for A/B testing). Gradually, the old version disappears.So, in addition to labels, play with replicas, too.
Chapter 12 - Helm
Helm is a templating engine and package manager for a set of K8s manifests.
Chart-file: Bundles manifests that comprise the API resources of an application.
Artifact Hub: helm executables online.
helm repo list 
	helm repo add http …
helm search repo jenkinsci (--versions)
helm install my-jenkins jenkinci/jenkins --version 4.6.4
Chart automatically created K8s objects
--values : overrides from yaml file
--set : overrides directly from command line
To discover config options
helm show values jenkinsci/jenkins
-n option works in helm install
helm list --all-namespaces
helm repo update & helm upgrade my-jenkins jenkinsci/jenkins --version 4.6.5 (To change values, use --set/--values again)
helm uninstall my-jenkins
Chapter 13 - API Deprecations
kubectl api-versions
Keep the deprecated API migration guide page handy to tackle such scenarios.
Chapter 14 - Container Probes
A health probe is a periodically running mini-process that asks the application for its status and takes action upon certain conditions.
Readiness probe checks if the application is ready to serve incoming requests.
Liveness probe checks for the application’s responsiveness.
Startup probe is a check before liveness probe starts.
Health verification methods:
exec command: executes a command inside the container and checks its exit code.
httpGet: sends an HTTP get request to an endpoint exposed by the application.
tcpSocket: Tries to open a TCP socket connection to a port.
gRPC: app implements gRPC health-checking protocol.
Health check attributes:
initialDelaySeconds (0): Delay in seconds until first check is executed
periodSeconds (10): interval for executing a check.
timeoutSeconds (1): max # of secs until the check operation timed out.
successThreshold (1): # of successful check attempts until the probe is considered successful after a failure.
failureThreshold (3): # of failures for check attempts before the probe is marked failed and takes action.
terminationGracePeriodSeconds(30): Grace period before forcing a container to stop upon failure.
The kubelet puts the readiness and liveness probes on hold while the startup probe is running.
Chapter 15 - Troubleshooting Pods and Containers
If the number of restarts is greater than zero, one might want to check the logic of the liveness probe and identify the reason. 
Common error conditions
ImagePullBackOff or ErrImagePull
CrashLoopBackOff : Application or the command run in container crashes.
CreateContainerConfigError: ConfigMap or Secret referenced by container cannot be found.
If no errors, kubectl describe pod.. => kubectl get events … (Lists the events across all Pods for a given namespace) => kubectl port-forward <pod-name> 2500:80 (This command forwards HTTP connections from a local port to a port exposed by a Pod).
In kubectl log command -f option streams logs and --previous gets the logs from prev instantiation of container.
Then open an interactive shell to a container.
One can deploy an ephemeral container for troubleshooting minimal containers, like distroless.
A raw debug command: inject an ephemeral container to a running pod for debugging purposes.
Examples of typical K8s metrics:
# of nodes in the cluster
health status of nodes
node performance metrics like CPU, memory, disk space, etc.
pod-level performance metrics
kubelet sends metrics from nodes to metrics server but metrics server does not persist data over time.
Chapter 16 - CRDs
A Custom Resource Definition is a K8s extension mechanism for introducing custom API primitives to fulfill requirements not covered by built-in primitives.
CRD is just a schema, to make it useful, a CRD has to be backed by a controller.
CRD + controller => operator pattern
For the exam, one will need to understand how to discover CRD schemas provided by external operators and how to interact with objects that follow the CRD schema.
CRD specifies the group, version and names of the custom primitive.
It also spells out all the “configurable” attributes including their data types.
kubectl api-resources --api-group=<some api group>
kubectl get crds
Chapter 17 -Athentication, Authorization and Admission Control
User or Service Control -> Authentication - Authorization - Admission Control -> Process
Authentication: Client certificates or Bearer Tokens
Authorization: Web + HTTP request path 
Standard K8s RBAC model is utilized.
List Pods/ create a new service object etc.
Admission control verifies if the request is well formed and potentially needs to be modified before the request is processed.
The kubeconfig file defines API server endpoints of the clusters we want to interact with, and a list of users registered with the cluster including their credentials in the form of client certificates.
The mapping between a cluster and user for a given namespace is called “context”.
kubectl config view
kubectl config current-context
kubectl config use-context bmuschko
kubectl config set-credentials myuser --client-key=myuser.key --client-certificate=myuser.crt --embed-certs=true
RBAC defines policies for users, groups service accounts by allowing/disallowing access to manage API resources.
subject examples: groups, users, service accounts
API resources: configmap, pod, deployment, node, etc.
operations: create, list, watch, delete, etc.
subjects, API resources and operations are the building blocks of RBAC key.
The Role API primitive declares the API resources and their operations this rule should operate on in a specific namespace.
The RoleBinding binds the Role object to the subject(s) in a specific namespace.
Namespace-wide RBAC vs. cluster-wide RBAC
Default roles(user-facing roles):
cluster-admin: R/W across all namespaces.
admin: R/W access in namespace including Role and RoleBinding
edit: RW in namespace except Role & RoleBinding . Provides access to secrets.
view: Read only access in ns except Roles and RoleBindings and Secrets.
Custom node creation:
	kubectl create role --verb=list,get,watch --resource=pods,deployments,services (* for meaning all) --resource-name=<resource name> (optional)
kubectl get roles
kubectl describe role read-only
kubectl create rolebinding read-only-binding --role=role-binding --user(--group, --serviceaccount)=bmuschko
kubectl auth can-i --list --as murat
Pods can use a service account to authenticate with the API server through an auth token (or CI/CD pipeline can use serviceaccount).
Pod uses the ”default” service account.
Create service account => create role => create role binding

Admission controller provides a way to approve, deny or mutate a request before it takes effect.
kube-apiserver --enable-admission-plugins=NamespaceLifecycle,PodSecurity,LimitRanger
Chapter-18 Resource Requirements, Limits and Quotas
The ResourceQuota defines aggregate resource constraints on ns-level.
A LimitRange is  a policy that constrains or defaults the resource allocations for a single object of a specific type.
ResourceRequests are defined by the containers in a pod.
spec.containers[].resources.requests.cpu (500m)
spec.containers[].resources.requests.memory(64Mi)
spec.containers[].resources.requests.hugepages-<size> (hugepages-2Mİ=60Mi)
spec.containers[].resources.requests.ephemeral-storage=4Gi
Resource limits ensure that the container cannot consume more than the allotted resource amounts.
Same in resource request in definition except not requests but limits.
Rules that can be defined:
Setting an upper limit for the # of objects.
Limiting the total sum of compute resources.
Expecting a Quality of Service (QoS) class for a Pod.
The enforcement of LimitRange rules happens during the admission control phase when processing an API request.
It is advisable to only create a single LimitRange object per namespace.
spec.limits.defaultRequest.(cpu): default request value for container
spec.limits.default. : default limit for container
spec.limits. min/max: min or max value for resource request on limit
Chapter 19 - ConfigMaps and Secrets
The cluster component that stores data of a ConfigMap and Secret object is etcd.
Options for kubectl create configmap
--from-literal=locale=en_US
--from-env-file=config.env
--from-file=app-config.json (properties/YAML file)
--from-file=config dir
Injecting configmap as environment variable to containers: spec.containers[].envFrom[].configMapRef
One can inject configMap as volume, too.
kubectl create secret <secret type> name <options>
secret types and internal types:
generic => Opaque
docker-registry => kubernetes.io/dockercfg
tls => kubernetes.io/tls
Reading secrets is similar to reading environment variables.
There are specialized secret types: ex. kubernetes.io/basic-auth 
While consuming secrets, use spec.containers[].env[].valueFrom.secretKeyRef (redefining keys)
Mounting secrets is also possible.
Chapter 20 - Security Contexts
A security context defines privilege and access control settings for containers as part of a Pod specification.
The security context is not a K8s primitive. It is modeled as a set of attributes under the directive “securityContext” within the Pod specification. (Pod-level vs container-level).
spec.securityContext vs. spec.containers[].securityContext
In deployment’s Pod template section, the same syntax can be used.
Pod security admission is possible.
One can restrict pods by user via USER instruction in Dockerfile.
securityContext.runAsNonRoot:(true/false)
Chapter 21 - Services
The Service implements an abstraction layer on top of Pods, assigning a fixed virtual IP fronting all Pods with matching labels.
Similar to a Deployment, the Service determines the Pods it works on with the help of label selection.
Service: spec.selector: tier:frontend
Pod1: spec.selector: tier:backend
Service types: 
Cluster IP: Only reachable inside the cluster. Round-robin to distribute traffic evenly among Pods.
NodePort: Accessible from outside of the cluster. Expose service on each node’s IP address at a static port.
LoadBalancer: Exposes the Service externally using a cloud provider’s load Balancer
The ClusterIP Service type is suitable for use cases that call for exposing a microservice to other Pods within the cluster.
NodePort usage is problematic in some ways: 
Node port (=port of a node) is allocated dynamically.
Traffic goes to only one node (with given IP address), so no load balancing
Publicly available node port is a vulnerability.
So, only for testing/development purposes.
LoadBalancer is matched with an external load balancer IP.
	Network traffic will be distributed across nodes.
	But it can be expensive, so Ingress can be used.
For a successful traffic forwarding, not only label selection but also port mapping with Pods.
Service accepts traffic at “spec.ports[].port”, routes towards spec.ports[].targetPort
The target port has the same value as spec.ports[].containerPort in the Pods.
kubectl create service clusterip echoserver --tcp=80:8080
To create a Pod and corresponding Service at the same time:
	kubectl run echoserver --image=<image name> --restart=Never --port=8080 --expose
kubectl expose deployment echoserver --port=80: --target-port=8080
An endpoint is a resolvable network endpoint, aka Virtual IP address and container port of a Pod.
CoreDNS will store the Service name as a hostname and map it to the ClusterIP address.
Referencing just the hostname of the Service does not work across namespaces. You need to append the namespace as well:
	<service name>.<namespace>:<port>
Full hostname: echoserver.default.svc.cluster.local
<SERVICE_NAME>_SERVICE_HOST and <SERVICE_NAME>_SERVICE_PORT environment variables exist inside apps in the related pod.
NodePort is created very similar to cluster ip.
From within the cluster, you can still access the Service using the cluster ip address and port number.
	To find the node’s ip address
Render node details
status.hostIP attribute of POd
spec.loadBalancer: The external IP address assigned to the Service at run-time.
Chapter 22 - Ingresses
The Ingress exposes HTTP(s) routes to clients outside of the cluster through an externally reachable URL. The routing rules configured with the Ingress determine how the traffic should be routed.
For ingress to function, an Ingress controller is essential. This controller assesses the set of rules outlined by Ingress, dictating the routing of traffic.
Ex: Nginx Ingress controller, AKS Application Gateway Ingress Controller

Find at least one Pod which runs the Ingress controller.
spec.ingressClassName: the selection of a specific controller implementation by name.
Rule criteria for ingress:
An optional host
A list of paths
The backend (Service_name:port)
kubectl create ingress next-app --rule=”next-example.com/app=app-service:8080” …
Creating with YAML has one major difference: the assignment of an Ingress controller annotation.
spec.rules[].http.paths[].pathType can have two values: Exact or Prefix. Difference between them is trailing slash
To enable the routing of HTTP(s) traffic through the Ingress and subsequently to the configured Service, it is crucial to set up a DNS entry mapping to the external address. This typically involves configuring either an A record or CNAME record.
Chapter 23 - Network Policies
Within a K8s cluster, any Pod can talk to any other Pod without restrictions using its IP address or DNS name, even across namespaces. 
	A network policy defines rules that control traffic from and to a Pod
The network policy controller evaluates the collection of rules defined by a network policy(Cilium).
In the context of a network policy, incoming traffic is called “ingress”, and outgoing traffic is called “egress”. For ingress and egress, you can whitelist the services of traffic like Pods, IP addresses, or Ports.
Network policies do not involve Services at all. All rules are ns- or Pod-specific.
A network policy cannot be defined with imperative create command but YAML file.
Spec-attributes of network policy includes: 
podSelector
policyTypes: Type of traffic (ingress and/or egress)
ingress: list of rules for incoming traffic (from, ports) 
ingress: list of rules for outgoing traffic (to, ports)
Attributes of a network policy to and from selectors:
podSelector
nameSpaceSelector
nameSpaceSelector & podSelector


