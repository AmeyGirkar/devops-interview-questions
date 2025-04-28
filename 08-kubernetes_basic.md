Of course! Here’s 50 Kubernetes interview questions with simple and bullet-point answers (easy to revise):

⸻

1. What is Kubernetes?
	•	Open-source container orchestration tool.
	•	Manages deployment, scaling, and operations of containerized applications.

2. What is a Pod?
	•	Smallest unit in Kubernetes.
	•	Holds one or more containers.

3. What is a Node?
	•	Machine (VM or physical) where Pods run.

4. What is a Cluster?
	•	Group of nodes managed by Kubernetes.

5. What is Kubelet?
	•	Agent running on each node.
	•	Communicates with Kubernetes Control Plane.

6. What is etcd?
	•	Key-value store.
	•	Stores cluster data.

7. What is Control Plane?
	•	Manages the cluster.
	•	Components: API Server, Controller Manager, Scheduler, etcd.

8. What is Kube-proxy?
	•	Handles networking for Pods.
	•	Maintains network rules.

9. What is the API Server?
	•	Front-end for Kubernetes.
	•	Handles REST operations.

10. What is a ReplicaSet?
	•	Ensures specified number of Pod replicas are running.

⸻

11. What is a Deployment?
	•	Manages ReplicaSets.
	•	Used for rolling updates and rollbacks.

12. What is a Service?
	•	Exposes a set of Pods over a network.

13. Types of Services in Kubernetes?
	•	ClusterIP (default)
	•	NodePort
	•	LoadBalancer
	•	ExternalName

14. What is ConfigMap?
	•	Stores configuration data as key-value pairs.

15. What is Secret?
	•	Stores sensitive data (e.g., passwords).

16. What is a Namespace?
	•	Virtual cluster inside Kubernetes.
	•	Separates resources.

17. What is a StatefulSet?
	•	Manages stateful applications.
	•	Provides stable identity and storage.

18. What is a DaemonSet?
	•	Runs a copy of a Pod on all (or some) nodes.

19. What is a Job?
	•	Creates Pods to run a task until it completes.

20. What is a CronJob?
	•	Schedules Jobs to run periodically.

⸻

21. What is Ingress?
	•	Manages external access to services.
	•	Acts like a smart router.

22. What is a Helm Chart?
	•	Package manager for Kubernetes applications.

23. What is Horizontal Pod Autoscaler (HPA)?
	•	Automatically scales Pods up/down based on CPU/memory usage.

24. What is Vertical Pod Autoscaler (VPA)?
	•	Automatically adjusts Pod CPU/memory requests.

25. What is Cluster Autoscaler?
	•	Adds/removes nodes based on load.

26. What is a Persistent Volume (PV)?
	•	Piece of storage provisioned for a Pod.

27. What is a Persistent Volume Claim (PVC)?
	•	Request for storage by a Pod.

28. Difference between Deployment and StatefulSet?
	•	Deployment: for stateless apps.
	•	StatefulSet: for stateful apps (like databases).

29. How do you perform a rolling update in Kubernetes?
	•	Update Deployment object with a new image version.

30. What is a RollingUpdate strategy?
	•	Gradually replace old Pods with new ones.

⸻

31. What is a Blue-Green Deployment?
	•	Two environments (blue: old, green: new).
	•	Switch traffic to green after testing.

32. What is Canary Deployment?
	•	Gradually shift traffic to the new version.

33. What is Node Affinity?
	•	Schedules Pods to preferred nodes.

34. What is Taint and Toleration?
	•	Taints: repel Pods from nodes.
	•	Tolerations: allow Pods to run on tainted nodes.

35. What is a ResourceQuota?
	•	Limits resource usage in a namespace.

36. What is a LimitRange?
	•	Sets default resource limits/requests.

37. How does Kubernetes handle secrets securely?
	•	Encrypts Secrets at rest (optional).
	•	Access control using RBAC.

38. What is Kubernetes Dashboard?
	•	Web UI to manage cluster resources.

39. What is RBAC in Kubernetes?
	•	Role-Based Access Control.
	•	Manages permissions.

40. How to update a running Pod?
	•	Update the Deployment (Pods are recreated).

⸻

41. Can you edit a Pod directly?
	•	No, Pods are immutable.
	•	Instead, update Deployment or recreate Pod.

42. What happens when a Pod crashes?
	•	Controller (e.g., Deployment) automatically creates a new Pod.

43. What is Init Container?
	•	Container that runs before the main app container starts.

44. What is a Sidecar Container?
	•	Secondary container in a Pod that supports the main container.

45. How do you troubleshoot a Pod?
	•	kubectl logs pod-name
	•	kubectl describe pod pod-name

46. What is a Volume in Kubernetes?
	•	Directory accessible to containers in a Pod.
	•	For data sharing and persistence.

47. What is a Pod Disruption Budget (PDB)?
	•	Limits voluntary disruptions to Pods.

48. What is Eviction in Kubernetes?
	•	Kubernetes removes Pods when nodes are under resource pressure.

49. How do you expose a service externally?
	•	Use NodePort, LoadBalancer, or Ingress.

50. How do you monitor a Kubernetes cluster?
	•	Tools like Prometheus, Grafana, or native Kubernetes Metrics Server.

⸻

Would you also like me to give you a PDF file for easy revision? 📄✨
(If yes, I’ll quickly prepare it for you!)
