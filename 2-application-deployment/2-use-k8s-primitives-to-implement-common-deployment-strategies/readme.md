# Test yourself tasks

Complete the following tasks in your own time.

## Restrictions

The following restrictions will help emulate exam conditions.

- Only use approved websites (`kubernetes.io/docs`, `kubernetes.io/blog`, `github.com/kubernetes` and `helm.sh`)
- Use only your chosen product's man pages and other documentation installed by your products (E.g. `buildah --help`, `docker --help`, `podman --help`)
- Do not refer to any other notes
- Do not make any hand-written notes

The full list of exam requirements and restrictions can be found [here](https://docs.linuxfoundation.org/tc-docs/certification/lf-candidate-handbook/exam-rules-and-policies).

## Pre-requisites

The images and containers you'll work with are all Linux-based. It's recommended to test yourself in a Linux-based environment. This is because the exam environment is Linux-based.

You can have any of the following tools installed for use.

- Buildah
- Docker
- Podman

If you haven't already done so, clone the repo with the following command.

```
$ git clone https://github.com/nigelpoulton/ckad.git
```

Run the following command to change into the correct directory. The tasks expect your terminal to be in this directory. If it isn't, the answers won't work.

```
$ cd ckad/2\ Application\ Deployment/2\ Use\ Kubernetes\ Primitives\ to\ Implement\ Common\ Deployment\ Strategies/
```

**NOTE:** This will set a very long working directory. If this bothers you (it bothers me) you can tell Linux just to display your current directory without the path by running this command. It will only change your prompt for your current shell session.

```
$ PS1="\W\$ "
```

## Answers

Answers to the question tasks can be found in the `answers.md` file.

## Questions

### Task 1

1. Create a Deployment named `nginx-deploy`. The Deployment should use the `nginx:stable-alpine` image and create `4` replicas.

2. Create a `NodePort` service named `nginx-svc` and associate it with the Pods created by the `nginx-deploy` Deployment. The service should expose port `9000` and a target port of 80.

3. Edit the `nginx-deploy` Deployment and change the number of replicas to `6`.

4. Ensure the service is associated with the `nginx-deploy` Pods by running the following command and checking that it returns HTML content. Replace `<node-port-value>` with the `nginx-svc` node port value.

`curl localhost:<node-port-value>`

JBUI:
We can't do this from our "own" machine (WSL or Windows) as our control plane node does not share our own localhost network.
Although it is possible to let a docker container share the localhost network: https://docs.docker.com/engine/network/drivers/host/
This is often not desired, we rather explicitly map available host ports (from our WSL or windows machine) to the ports of a docker container in our case the kind cluster.
For now to be able to run the curl command above open a shell from within the control-plane node, as this will give you access to the localhost network of the control-plane node.

### Task 2

**JBUI: IMPORTANT:**
We will need to do some additional set-up to get the Service type LoadBalancer to work with our Kind set-up.
A use-case for the LoadBalancer type in Kubernetes is to provision an External ip address which resolves to the service.
Kubernetes itself generally does not have the capability to do this provisioning on it's own.
For example in Azure when we create such a service with LoadBalancer type, Azure will pick this up and provision a public ip address resource for us and bind it to the service.
This allows us to reach the service via the newly provisioned ip address.
For our local kind cluster we need to play the role of Azure and provision these external IP addresses in our own machines network to the LoadBalancer services.
Luckily there is an out of the box solution for kind: https://kind.sigs.k8s.io/docs/user/loadbalancer/
Following the docs run the install of the go binary from your WSL.
```
go install sigs.k8s.io/cloud-provider-kind@latest
```
And from a separate terminal launch the application and keep it running when working with your kind cluster.
This one will check all available local kind clusters, and within those clusters bind all found LoadBalancer Services to an ip address which is reachable from your machine.
```
cloud-provider-kind
```


**IMPORTANT:** This task assumes that you have images named `ckad:blue` and `ckad:green` available on your system. To create them, go to the `Blue-Green/build` folder and run `docker-compose build` to create the images.

JBUI:
Remember we are using our dedicated kind hosted OCI registry.

The current folder contains Kubernetes YAML files to create Blue/Green deployments and services. Images you'll use are also available on the system. View the files in the current folder using:

`ls -l`

1. View the selectors in the `blue.svc.yml` and `green.svc.yml` files. Add a `role: blue` selector to `public.svc.yml`.
JBUI:
And fix the issue if your pods are unable to come online

2. Create the resources in Kubernetes using the YAML files available in the current folder. Run the following command to verify that the `blue` Pods are accessible using the public service.

JBUI:
Check which ip address the LoadBalancer got assigned to by inspecting your services and performing a curl, or check via the browser!

    `curl {{EXTERNAL_IP}}`

3. Change the public service's `role` selector from `blue` to `green`. Run the following command to verify that the `green` Pods are accessible using the public service.

    `curl {{EXTERNAL_IP}}`

### Task 3

**IMPORTANT:** This task assumes that you have images named `ckad:stable` and `ckad:canary` available on your system. To create them, go to the `Canary/build` folder and run `docker-compose build` to create the images.

Your system has `ckad:stable` and `ckad:canary` images available and the current folder contains YAML files to be used to complete the task.

1. Create resources in Kubernetes using the YAML files available in the current folder. List all the running Pods.

2. Scale the `stable` Deployment to `6` replicas and the `canary` Deployment to `2` replicas.

3. Modify the `canary` Deployment so that the Service will match it and direct traffic to the Pods.

4. Create a temporary Pod named `temp-pod` as an interactive TTY. Use the `alpine` image for the Pod and set `restart` to `Never`.

5. Install `curl` into the `temp-pod` container using `apk add curl`, and run the following command until the output from a `canary` Pod is displayed:

    `curl {{CLUSTER_IP_PUBLIC_SVC}}`
    or
    `curl public-service`

