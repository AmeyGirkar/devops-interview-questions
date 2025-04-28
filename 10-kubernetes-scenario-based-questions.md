# Kubernetes Troubleshooting Scenarios: 20 Questions with Answers

## 1. Image Pull Error: Private Registry Access

**Scenario:** Your team has deployed a new application using a Deployment, but all pods are stuck in `ImagePullBackOff` status. The application image is hosted in your company's private Docker registry.

**Question:** What is the most likely cause of this issue and how would you fix it?

**Answer:**
- **Cause:** Missing or invalid image pull secrets for the private registry.
- **Solution:**
  1. Create a Kubernetes secret with registry credentials:
     ```
     kubectl create secret docker-registry regcred \
       --docker-server=<registry-url> \
       --docker-username=<username> \
       --docker-password=<password> \
       --docker-email=<email>
     ```
  2. Update the deployment to use the secret:
     ```yaml
     spec:
       template:
         spec:
           imagePullSecrets:
           - name: regcred
     ```
  3. Apply the updated deployment:
     ```
     kubectl apply -f deployment.yaml
     ```

## 2. Crashing Pods: Resource Constraints

**Scenario:** After deploying a memory-intensive application, you notice the pods continuously crash and restart with status `CrashLoopBackOff`.

**Question:** How would you diagnose and resolve this issue?

**Answer:**
- **Diagnosis:**
  1. Check pod logs:
     ```
     kubectl logs <pod-name>
     ```
  2. Look for OOMKilled messages:
     ```
     kubectl describe pod <pod-name>
     ```

- **Solution:**
  1. Increase memory limits in deployment:
     ```yaml
     resources:
       requests:
         memory: "512Mi"
       limits:
         memory: "1Gi"
     ```
  2. Apply the configuration:
     ```
     kubectl apply -f deployment.yaml
     ```
  3. Alternatively, patch existing deployment:
     ```
     kubectl patch deployment <name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","resources":{"limits":{"memory":"1Gi"}}}]}}}}'
     ```

## 3. Pending Pods: Insufficient Resources

**Scenario:** After scaling your application to 10 replicas, you notice 4 pods are stuck in `Pending` status.

**Question:** What could be causing this issue and how would you investigate?

**Answer:**
- **Cause:** Insufficient cluster resources (CPU, memory) to schedule all pods.
- **Investigation:**
  1. Check pod status and events:
     ```
     kubectl describe pod <pending-pod-name>
     ```
  2. Check node resource usage:
     ```
     kubectl describe nodes
     ```
  3. Look for scheduler events:
     ```
     kubectl get events --sort-by='.lastTimestamp'
     ```
- **Solution:**
  1. Adjust resource requests to be more reasonable
  2. Scale down the deployment temporarily:
     ```
     kubectl scale deployment <name> --replicas=6
     ```
  3. Add more nodes to the cluster if possible

## 4. Case of the Missing Pods: Namespace Confusion

**Scenario:** A developer reports that their recently deployed application is missing after running `kubectl get pods`, but they're confident it was deployed successfully.

**Question:** What is a likely explanation and how would you help them find their pods?

**Answer:**
- **Cause:** The pods were deployed to a different namespace than expected.
- **Solution:**
  1. Check pods across all namespaces:
     ```
     kubectl get pods --all-namespaces
     ```
  2. Look for pods with specific labels:
     ```
     kubectl get pods --all-namespaces -l app=<app-name>
     ```
  3. Verify the current namespace context:
     ```
     kubectl config get-contexts
     ```
  4. Switch to the correct namespace:
     ```
     kubectl config set-context --current --namespace=<correct-namespace>
     ```

## 5. Schrodinger's Deployment: Stuck Rollout

**Scenario:** You've updated your application image tag and applied the changes, but the rollout seems stuck with both old and new pods running. The deployment shows status as "progressing" for an unusually long time.

**Question:** What could be preventing the deployment rollout from completing?

**Answer:**
- **Cause:** Failed readiness probes in new pods preventing the rollout from completing.
- **Investigation:**
  1. Check rollout status:
     ```
     kubectl rollout status deployment/<name>
     ```
  2. Examine pod readiness:
     ```
     kubectl get pods
     ```
  3. Check pod events and logs:
     ```
     kubectl describe pod <new-pod-name>
     kubectl logs <new-pod-name>
     ```
- **Solution:**
  1. Fix the readiness probe configuration or underlying application issue
  2. Or roll back to the previous version:
     ```
     kubectl rollout undo deployment/<name>
     ```

## 6. Create Container Errors: Missing ConfigMap

**Scenario:** Your application pods are failing to start with `CreateContainerError` status.

**Question:** What steps would you take to diagnose and fix this issue?

**Answer:**
- **Diagnosis:**
  1. Check pod details:
     ```
     kubectl describe pod <pod-name>
     ```
  2. Look for errors mentioning missing volumes or configs
  
- **Cause:** The pod is trying to mount a ConfigMap that doesn't exist
- **Solution:**
  1. Create the missing ConfigMap:
     ```
     kubectl create configmap <config-name> --from-file=config.json
     ```
  2. Verify it exists:
     ```
     kubectl get configmap <config-name>
     ```
  3. Restart the deployment:
     ```
     kubectl rollout restart deployment/<name>
     ```

## 7. Config Out of Date: Stale Configuration

**Scenario:** You've updated a ConfigMap with new values, but your application is still using the old configuration even after waiting for some time.

**Question:** Why isn't your application picking up the new configuration and how would you fix it?

**Answer:**
- **Cause:** Kubernetes doesn't automatically restart pods when ConfigMaps change.
- **Solution:**
  1. Restart the deployment to pick up new ConfigMap:
     ```
     kubectl rollout restart deployment/<name>
     ```
  2. For future, implement a ConfigMap reload strategy:
     - Use a sidecar container that watches for changes
     - Add a hash of the ConfigMap in pod annotations
     - Use a tool like Reloader

## 8. Reloader: Automatic Config Updates

**Scenario:** Your team frequently updates ConfigMaps and Secrets, but is tired of manually restarting deployments each time.

**Question:** How would you implement automatic reloading of pods when ConfigMaps or Secrets change?

**Answer:**
- **Solution:**
  1. Install Reloader (a Kubernetes controller):
     ```
     kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml
     ```
  2. Add annotations to your deployments:
     ```yaml
     metadata:
       annotations:
         configmap.reloader.stakater.com/reload: "my-configmap"
         secret.reloader.stakater.com/reload: "my-secret"
     ```
  3. Now pods will automatically restart when the referenced ConfigMap/Secret changes

## 9. Endlessly Terminating Pods: Stuck Finalizers

**Scenario:** You've tried to delete a pod, but it remains in `Terminating` state for an unusually long time (over 30 minutes).

**Question:** What could be causing this and how would you force the pod deletion?

**Answer:**
- **Cause:** Stuck finalizers that prevent the pod from being deleted.
- **Solution:**
  1. Check pod status:
     ```
     kubectl describe pod <pod-name>
     ```
  2. Force delete the pod by removing finalizers:
     ```
     kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge
     ```
  3. As a last resort, force delete with grace period of 0:
     ```
     kubectl delete pod <pod-name> --grace-period=0 --force
     ```

## 10. Field Immutability: Service IP Changes

**Scenario:** You're trying to update a Service to use a specific cluster IP, but receive an error message stating "field is immutable".

**Question:** How can you change the cluster IP of an existing service?

**Answer:**
- **Cause:** Cluster IP field is immutable after creation.
- **Solution:**
  1. Export the current service definition:
     ```
     kubectl get service <service-name> -o yaml > service.yaml
     ```
  2. Modify the YAML file to include the desired cluster IP
  3. Delete the existing service:
     ```
     kubectl delete service <service-name>
     ```
  4. Create the new service with modified YAML:
     ```
     kubectl apply -f service.yaml
     ```

## 11. enableServiceLinks: Environment Variable Overload

**Scenario:** Your application pods have an extremely large number of environment variables, causing startup delays and configuration confusion.

**Question:** What might be causing this environment variable explosion and how can you reduce them?

**Answer:**
- **Cause:** By default, Kubernetes injects all service information as environment variables (`enableServiceLinks: true`).
- **Solution:**
  1. Disable service links in your pod spec:
     ```yaml
     spec:
       enableServiceLinks: false
     ```
  2. Apply the updated configuration:
     ```
     kubectl apply -f deployment.yaml
     ```
  3. Use DNS for service discovery instead of environment variables

## 12. Interns Can See Our Secrets: RBAC Issues

**Scenario:** You've discovered that interns (with limited access) can view sensitive production secrets through the Kubernetes API.

**Question:** How would you diagnose and fix this permission issue?

**Answer:**
- **Diagnosis:**
  1. Check who can access secrets:
     ```
     kubectl auth can-i get secrets --as=intern-user
     ```
  2. Review role bindings:
     ```
     kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide | grep intern
     ```

- **Solution:**
  1. Create more restrictive roles:
     ```yaml
     kind: Role
     apiVersion: rbac.authorization.k8s.io/v1
     metadata:
       name: restricted-role
       namespace: production
     rules:
     - apiGroups: [""]
       resources: ["pods", "services"]
       verbs: ["get", "list", "watch"]
     ```
  2. Update role bindings for intern users:
     ```
     kubectl apply -f restricted-role.yaml
     kubectl apply -f new-rolebinding.yaml
     ```

## 13. Port Mania: Service Not Accessible

**Scenario:** You've deployed a service and pods, but cannot access the application through the service endpoint.

**Question:** What port-related misconfiguration could be causing this and how would you diagnose and fix it?

**Answer:**
- **Diagnosis:**
  1. Check service definition:
     ```
     kubectl describe svc <service-name>
     ```
  2. Verify endpoint connections:
     ```
     kubectl get endpoints <service-name>
     ```
  3. Test direct pod connectivity:
     ```
     kubectl exec -it <some-pod> -- curl <pod-ip>:<container-port>
     ```

- **Cause:** Mismatch between service port, targetPort, and container port
- **Solution:**
  1. Ensure service targetPort matches container port:
     ```yaml
     ports:
     - port: 80          # Service port
       targetPort: 8080  # Must match containerPort
     ```
  2. Check container port in deployment:
     ```yaml
     containers:
     - name: app
       ports:
       - containerPort: 8080  # Must match targetPort
     ```
  3. Apply corrected configuration:
     ```
     kubectl apply -f service.yaml
     kubectl apply -f deployment.yaml
     ```

## 14. Unreachable Pods: Network Policy Issues

**Scenario:** After implementing network policies, some microservices can no longer communicate with each other.

**Question:** How would you troubleshoot and fix this network connectivity issue?

**Answer:**
- **Diagnosis:**
  1. Check network policies:
     ```
     kubectl get networkpolicies -A
     ```
  2. Examine specific policy details:
     ```
     kubectl describe networkpolicy <policy-name>
     ```
  3. Test connectivity between pods:
     ```
     kubectl exec -it <pod-a> -- curl <service-b>
     ```

- **Solution:**
  1. Create or update network policy to allow required traffic:
     ```yaml
     kind: NetworkPolicy
     apiVersion: networking.k8s.io/v1
     metadata:
       name: allow-service-communication
       namespace: default
     spec:
       podSelector:
         matchLabels:
           app: service-a
       ingress:
       - from:
         - podSelector:
             matchLabels:
               app: service-b
         ports:
         - port: 80
           protocol: TCP
     ```
  2. Apply the updated policy:
     ```
     kubectl apply -f network-policy.yaml
     ```

## 15. What the Ingress: Application Inaccessible

**Scenario:** You've deployed an Ingress resource for your application, but users cannot access it through the defined hostname.

**Question:** What are the common causes and how would you troubleshoot this Ingress issue?

**Answer:**
- **Diagnosis:**
  1. Check Ingress status:
     ```
     kubectl describe ingress <ingress-name>
     ```
  2. Verify Ingress controller pods are running:
     ```
     kubectl get pods -n ingress-nginx
     ```
  3. Check backend service and endpoints:
     ```
     kubectl get svc <backend-service>
     kubectl get endpoints <backend-service>
     ```

- **Common causes:**
  - Ingress controller not deployed
  - Incorrect backend service name or port
  - Missing DNS entry for hostname
  - TLS certificate issues

- **Solution:**
  1. Ensure Ingress controller is running
  2. Update Ingress resource if needed:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: app-ingress
     spec:
       rules:
       - host: app.example.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: app-service  # Verify this exists
                 port:
                   number: 80       # Verify this is correct
     ```
  3. Set up proper DNS entry for hostname

## 16. Multi Attach Volume Errors: PVC Issues

**Scenario:** After a node failure, a replacement pod cannot be scheduled and shows errors about volume multi-attachment.

**Question:** What is happening and how would you resolve this issue?

**Answer:**
- **Cause:** The PersistentVolume is still attached to the failed node and can't be attached to a new node.
- **Diagnosis:**
  1. Check pod events:
     ```
     kubectl describe pod <pod-name>
     ```
  2. Look for multi-attach errors in events
  3. Check PV status:
     ```
     kubectl get pv <pv-name> -o yaml
     ```

- **Solution:**
  1. Delete the failed pod:
     ```
     kubectl delete pod <failed-pod-name> --grace-period=0 --force
     ```
  2. If still stuck, manually detach the volume by patching the PV:
     ```
     kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'
     ```
  3. Then re-create the pod or scale the deployment:
     ```
     kubectl scale deployment <name> --replicas=0
     kubectl scale deployment <name> --replicas=1
     ```

## 17. Container Startup Crash: Command & Arguments

**Scenario:** Your containerized application keeps crashing immediately after starting, showing `CrashLoopBackOff` status with exit code 127.

**Question:** What is likely causing this issue and how would you troubleshoot it?

**Answer:**
- **Cause:** Exit code 127 typically indicates "command not found" - the container entrypoint or command is invalid.
- **Diagnosis:**
  1. Check pod logs:
     ```
     kubectl logs <pod-name>
     ```
  2. Look at pod description:
     ```
     kubectl describe pod <pod-name>
     ```
  3. Review deployment command/args:
     ```
     kubectl get deployment <name> -o yaml
     ```

- **Solution:**
  1. Correct the command and args in deployment:
     ```yaml
     containers:
     - name: app
       command: ["/bin/sh"]
       args: ["-c", "valid-executable param1 param2"]
     ```
  2. Apply the updated configuration:
     ```
     kubectl apply -f deployment.yaml
     ```
  3. Verify by checking new pod logs

## 18. Persistent Storage Issues: Incorrect Storage Class

**Scenario:** A new persistent volume claim (PVC) remains in `Pending` state and doesn't provision a volume.

**Question:** What could be causing this issue and how would you resolve it?

**Answer:**
- **Diagnosis:**
  1. Check PVC status:
     ```
     kubectl describe pvc <pvc-name>
     ```
  2. Look for events related to storage provisioning
  3. Check available storage classes:
     ```
     kubectl get storageclasses
     ```

- **Causes:**
  - Referenced storage class doesn't exist
  - No default storage class set
  - Storage provisioner unavailable

- **Solution:**
  1. Specify an existing storage class:
     ```yaml
     kind: PersistentVolumeClaim
     apiVersion: v1
     metadata:
       name: data-pvc
     spec:
       storageClassName: standard  # Use an existing class
       accessModes:
         - ReadWriteOnce
       resources:
         requests:
           storage: 10Gi
     ```
  2. Or set a default storage class:
     ```
     kubectl patch storageclass <name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
     ```

## 19. Node Cordoned: Pods Not Scheduling

**Scenario:** After deploying a new StatefulSet, you notice new pods remain in `Pending` state, and all of them are trying to schedule on a specific node.

**Question:** What might be causing this scheduling issue and how would you resolve it?

**Answer:**
- **Diagnosis:**
  1. Check pod status:
     ```
     kubectl get pods -o wide
     ```
  2. Examine pod events:
     ```
     kubectl describe pod <pod-name>
     ```
  3. Check node status:
     ```
     kubectl describe node <node-name>
     ```

- **Cause:** The node is cordoned (marked as unschedulable) or has taints
- **Solution:**
  1. Uncordon the node if appropriate:
     ```
     kubectl uncordon <node-name>
     ```
  2. Or remove specific taints:
     ```
     kubectl taint nodes <node-name> key:NoSchedule-
     ```
  3. If taints are required, add tolerations to your StatefulSet:
     ```yaml
     spec:
       template:
         spec:
           tolerations:
           - key: "key"
             operator: "Equal"
             value: "value"
             effect: "NoSchedule"
     ```

## 20. HorizontalPodAutoscaler Not Working: Missing Metrics

**Scenario:** You've configured a HorizontalPodAutoscaler (HPA) for your deployment, but it's not scaling despite increased load.

**Question:** What could be preventing the HPA from functioning correctly?

**Answer:**
- **Diagnosis:**
  1. Check HPA status:
     ```
     kubectl describe hpa <hpa-name>
     ```
  2. Look for "unable to fetch metrics" warnings
  3. Verify metrics server is running:
     ```
     kubectl get pods -n kube-system | grep metrics-server
     ```

- **Causes:**
  - Missing metrics server
  - Insufficient resource requests set on pods
  - Incorrect metric configuration

- **Solution:**
  1. Install metrics server if missing:
     ```
     kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
     ```
  2. Ensure pods have resource requests defined:
     ```yaml
     resources:
       requests:
         cpu: 100m
         memory: 128Mi
     ```
  3. Verify HPA configuration:
     ```yaml
     apiVersion: autoscaling/v2
     kind: HorizontalPodAutoscaler
     metadata:
       name: app-hpa
     spec:
       scaleTargetRef:
         apiVersion: apps/v1
         kind: Deployment
         name: app-deployment
       minReplicas: 2
       maxReplicas: 10
       metrics:
       - type: Resource
         resource:
           name: cpu
           target:
             type: Utilization
             averageUtilization: 50
     ```
