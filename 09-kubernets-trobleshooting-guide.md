# Kubernetes Troubleshooting Guide

Here's a guide to common Kubernetes issues, their causes, and solutions:

## Image Pull Errors

**Causes:**
- Incorrect image name or tag
- Private registry requiring authentication
- Network connectivity issues
- Registry rate limiting

**Solutions:**
- Verify image name and tag: `kubectl describe pod <pod-name>`
- Check/update registry credentials:
  ```
  kubectl create secret docker-registry regcred --docker-server=<registry> --docker-username=<username> --docker-password=<password>
  ```
- Add imagePullSecrets to your deployment:
  ```
  kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "regcred"}]}'
  ```
- Check if registry is accessible from nodes

## Crashing Pods

**Causes:**
- Application errors
- Resource constraints (OOM)
- Configuration issues
- Init container failures

**Solutions:**
- Check logs: `kubectl logs <pod-name>` or `kubectl logs <pod-name> -p` for previous container
- Review events: `kubectl describe pod <pod-name>`
- Adjust resource limits: 
  ```
  kubectl set resources deployment <name> -c=<container> --limits=memory=<memory>,cpu=<cpu>
  ```
- Add liveness/readiness probes with appropriate delays

## Pending Pods

**Causes:**
- Insufficient cluster resources
- Node affinity/taints issues
- PVC binding issues
- Resource quota exceeded

**Solutions:**
- Check pod status: `kubectl get pods -o wide`
- Review events: `kubectl describe pod <pod-name>`
- Check node resources: `kubectl describe nodes`
- Verify PVC status: `kubectl get pvc`
- Check resource quotas: `kubectl describe quota`

## Case of the Missing Pods

**Causes:**
- Namespace issues
- Label selector mismatches
- Pods terminated by controllers
- Resource quotas

**Solutions:**
- Check all namespaces: `kubectl get pods -A`
- Verify deployment selectors: `kubectl describe deployment <name>`
- Review pod lifecycle events: `kubectl get events --sort-by='.lastTimestamp'`
- Check controllers: `kubectl get replicasets` or `kubectl get deployment`

## Schrodingers Deployment

**Causes:**
- Deployment stuck in intermediate state
- Conflicting resources
- Controller issues
- Update strategy problems

**Solutions:**
- Check deployment status: `kubectl rollout status deployment/<name>`
- Force rollout: `kubectl rollout restart deployment/<name>`
- Review deployment history: `kubectl rollout history deployment/<name>`
- Check events: `kubectl get events --field-selector involvedObject.kind=Deployment`

## Create Container Errors

**Causes:**
- Missing or incorrect volumes
- Permission issues
- Invalid container configuration
- Resource conflicts

**Solutions:**
- Check pod events: `kubectl describe pod <pod-name>`
- Verify volume configurations
- Check security contexts
- Review container startup commands
- Validate ConfigMaps and Secrets exist: `kubectl get configmap,secret`

## Config Out of Date

**Causes:**
- ConfigMaps or Secrets updated but not reflected in pods
- Volumes not updating
- Missing reload mechanisms

**Solutions:**
- Force pod recreation: `kubectl rollout restart deployment/<name>`
- Use ConfigMap/Secret hash in pod template annotations
- Implement a config watcher sidecar
- Use volume mounts for configs with `subPath` to enable updates

## Reloader

**Causes:**
- ConfigMaps or Secrets not triggering redeployments
- Missing config reload mechanism

**Solutions:**
- Add annotations to deployments for auto-reloading:
  ```
  annotations:
    configmap.reloader.stakater.com/reload: "my-configmap"
  ```
- Install a reloader operator: `kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml`
- Roll deployments manually: `kubectl rollout restart deployment/<name>`

## Endlessly Terminating Pods

**Causes:**
- Stuck finalizers
- Persistent volume claim issues
- Node problems
- Namespace termination stuck

**Solutions:**
- Force delete pod: `kubectl delete pod <pod-name> --grace-period=0 --force`
- Check PVC status: `kubectl get pvc`
- Remove finalizers: `kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge`
- Check node status: `kubectl get nodes`

## Field Immutability

**Causes:**
- Attempting to modify immutable fields
- Service IP changes
- Volume configuration changes

**Solutions:**
- Delete and recreate resource instead of updating
- Export current resource: `kubectl get <resource> <name> -o yaml > resource.yaml`
- Modify as needed, delete original, and recreate
- Use StatefulSets for stable identities

## enableServiceLinks

**Causes:**
- Environment variable conflicts
- Too many environment variables from services
- Performance issues from large number of service links

**Solutions:**
- Disable service links in pod spec:
  ```yaml
  enableServiceLinks: false
  ```
- Use DNS for service discovery instead of environment variables
- Specify only needed services in pod spec

## Interns can see our secrets

**Causes:**
- Overly permissive RBAC
- Secrets visible in logs or environment variables
- Lack of secret encryption

**Solutions:**
- Review and limit RBAC: `kubectl get rolebindings,clusterrolebindings`
- Create specific roles: `kubectl create role <role-name> --verb=get --resource=pods`
- Use external secret management (Vault, external-secrets operator)
- Encrypt secrets at rest: Configure etcd encryption
- Check who can access secrets: `kubectl auth can-i get secrets --as=<username>`

## Port Mania

**Causes:**
- Service port misconfigurations
- Incorrectly specified container ports
- Protocol mismatches
- Network policies blocking traffic

**Solutions:**
- Verify service configuration: `kubectl describe svc <service-name>`
- Check endpoint connections: `kubectl get endpoints <service-name>`
- Test connectivity from within cluster:
  ```
  kubectl run -it --rm debug --image=nicolaka/netshoot -- bash
  curl <service-name>:<port>
  ```
- Validate targetPort matches containerPort

## Unreachable Pods/Leaky Network Policies

**Causes:**
- Restrictive network policies
- CNI plugin issues
- Service CIDR conflicts
- Inter-node communication problems

**Solutions:**
- Check network policies: `kubectl get networkpolicies -A`
- Verify CNI is working: `kubectl get pods -n kube-system`
- Test pod connectivity: 
  ```
  kubectl exec -it <pod> -- ping <target>
  ```
- Create or update network policies:
  ```
  kubectl apply -f network-policy.yaml
  ```
- Check node network configuration

## What the Ingress

**Causes:**
- Misconfigured Ingress resource
- Ingress controller issues
- Certificate problems
- Backend service issues

**Solutions:**
- Check Ingress status: `kubectl describe ingress <name>`
- Verify Ingress controller is running: `kubectl get pods -n <ingress-controller-namespace>`
- Check TLS secrets: `kubectl get secret <tls-secret>`
- Validate backend services: `kubectl get svc <backend-service>`
- Test backend directly: `kubectl port-forward svc/<service> <local-port>:<service-port>`

## Multi Attach Volume Errors

**Causes:**
- PVC with ReadWriteOnce access mode mounted to multiple pods
- Node failure with volumes still attached
- Storage driver issues

**Solutions:**
- Check PVC access mode: `kubectl get pvc <pvc-name> -o yaml`
- Change access mode to ReadWriteMany if supported
- Use StatefulSets for stable PVC assignment
- Force detach volumes from failed nodes:
  ```
  kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'
  ```
- Check storage provider status

Would you like me to elaborate on any specific issue from this list?
