# Answers

The following lists each question and associated answer.

### Task 1

1. Create a Deployment named `nginx-deploy`. The Deployment should use the `nginx:stable-alpine` image and create `4` replicas.

```
k config set-context --current --namespace=dev
kubectl create deployment nginx-deploy --image=nginx:stable-alpine --replicas=4 --port=80
```

2. Create a `NodePort` service named `nginx-svc` and associate it with the Pods created by the `nginx-deploy` Deployment. The service should expose port `9000` and a target port of 80.

```
k expose deploy nginx-deploy --port=9000 --target-port=80 --type=NodePort --name=nginx-svc
```

3. Edit the `nginx-deploy` Deployment and change the number of replicas to `6`.

```
KUBE_EDITOR="nano" k edit deploy/nginx-deploy
```

4. Ensure the service is associated with the `nginx-deploy` Pods by running the following command and checking that it returns HTML content. Replace `<node-port>` with the `nginx-svc` node port value.

JBUI:
As the curl command will not work from our own WSL / Windows machine we first need to open a shell from the control-plane node.
Via the docker CLI this can be achieved with the following command:
```
docker exec -it {{KIND-CONTROL-PLANE CONTAINER ID OR NAME}} /bin/bash
```

```
k get service nginx-svc
curl http://localhost:<node-port-value>

```


### Task 2

**JBUI IMPORTANT:** This task assumes that you have images named `ckad:blue` and `ckad:green` available on from you OCI registry.
To create them you can do it multiple ways:
1. go to the `Blue-Green/Build` folder and run adjust the docker-compose to tag the image with the prefix image: localhost:5001/ckad:blue and image: localhost:5001/ckad:green. And push the images to the OCI registry.
2. go to the `Blue-Green/Build` folder and run `docker-compose build` to create the images. Tag them to localhost:5001 via the docker tag command and push them.
e.g.
```
docker tag ckad:blue localhost:5001/ckad:blue
docker push localhost:5001/ckad:blue
```
repeat for green

The current folder contains Kubernetes YAML files to create Blue/Green deployments and services. Images you'll use are also available on the system. View the files in the current folder using:

`ls -l`

1. View the selectors in the `blue.svc.yml` and `green.svc.yml` files. Add a `role: blue` selector to `public.svc.yml`.

JBUI:
Or vim

  ```
    nano blue.svc.yml
    nano green.svc.yml
    nano public.svc.yml
  ```

2. Create the resources in Kubernetes using the YAML files available in the current folder. Run the following command to verify that the `blue` Pods are accessible using the public service.

JBUI: heavily changed based on our Kind cluster

First find the External IP address:
`k get service`

This gives you something like:
```
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
blue-test-service    LoadBalancer   10.96.165.74   172.20.0.6    9000:31754/TCP   39m
green-test-service   LoadBalancer   10.96.79.43    172.20.0.5    9001:31700/TCP   39m
kubernetes           ClusterIP      10.96.0.1      <none>        443/TCP          18h
public-service       LoadBalancer   10.96.226.62   172.20.0.4    80:31111/TCP     39m
```
If External-IP is pending something might be wrong with your cloud-provider-kind, please sent a message to the Course teams chat.

And curl the external ip of the public-service
`curl {{EXTERNAL-IP}}`

3. Change the public service's `role` selector from `blue` to `green`. Run the following command to verify that the `green` Pods are accessible using the public service.

`curl {{EXTERNAL-IP}}`

Do it declarative via the public.svc.yml file
```
spec:
  type: LoadBalancer
  selector:
    app: nginx
    role: green
```
or imperative via:
```
k set selector svc public-service "role=green"
curl {{EXTERNAL-IP}}
```

### Task 3

**IMPORTANT:** This task assumes that you have images named `ckad:stable` and `ckad:canary` available on your system. To create them, go to the `Canary/build` folder and run `docker-compose build` to create the images.

Your system has `ckad:stable` and `ckad:canary` images available and the current folder contains YAML files to be used to complete the task.

1. Create resources in Kubernetes using the YAML files available in the current folder. List all the running Pods.

  ```
  k config set-context --current --namespace=dev
  k create –f ./
  ```

2. Scale the `stable` Deployment to `6` replicas and the `canary` Deployment to `2` replicas.

  ```
  k scale deploy stable-deployment –replicas=6
k scale deploy canary-deployment --replicas=2
  ```

3. Modify the `canary` Deployment so that the Service will match it and direct traffic to the Pods.

4. Create a temporary Pod named `temp-pod` as an interactive TTY. Use the `alpine` image for the Pod and set `restart` to `Never`.

  ```
  KUBE_EDITOR="nano" k edit deploy canary-deployment

  # Add app: customer-app label into deployment template labels
  ```


5. Install `curl` into the `temp-pod` container using `apk add curl`, and run the following command until the output from a `canary` Pod is displayed:

    ```
    # Get service cluster IP
    k get svc public-service
    k run -it --restart=Never --image=alpine temp-pod
    apk add curl
    ```
    `curl <service-cluster-ip>`
    or
    `curl public-service`

