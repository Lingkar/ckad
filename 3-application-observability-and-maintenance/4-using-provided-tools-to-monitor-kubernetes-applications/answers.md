# Answers

The following lists each question and associated answer.

### Task 1

1. Ensure that Metrics Server is installed and running.

    ```
    k get pods -n kube-system | grep metrics-server
    ```

JBUI:
Via the github link found in the k8s documentation, get the installation yaml resource for the metric server components:
```
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
In this file disable the ssl by adding the following argument to the metrics-server Deployment args.
```
--kubelet-insecure-tls
```


2. Create 10 nginx pods.

    ```
    kubectl create deployment nginx --image=nginx
    kubectl scale deployment nginx --replicas=10
    ```

3. Create a NodePort service for the Pods on port 80.

    ```
    k expose deployment nginx --port=80 --type=NodePort
    ```

4. Run a curl command against the Pods multiple times to generate traffic.

    ```
    k get svc nginx
    curl <EXTERNAL-IP>:<NODE_PORT>
    ```

5. View CPU and memory information for the Nodes and Pods.

    ```
    k top nodes
    k top pods
    ```

6. Get a raw view of a Node's CPU/memory.

    ```
    kubectl get --raw /api/v1/nodes/[node-name]/proxy/metrics/resource
    ```

