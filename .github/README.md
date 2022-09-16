# Ansible AWX on `minikube` macOS

We are running ...
* macOS Monterey 12.6
* on M1 Max chip with 64GB memory
* with Docker Desktop 4.11.1
* `minikube` 1.26.1

This document is based on Heikki Koivisto's ["Install AWX on mac" YouTube video](https://www.youtube.com/watch?v=yFWQBAPrWQE).
I ran into a few gotchas, so I decided to put up these notes. One notable difference is that Heikki uses the [`hyperkit`](https://github.com/moby/hyperkit#hyperkit) driver, whereas I am using docker. As of this writing, `hyperkit` does not have
M1 support.

### Start suitably-sized cluster

By default, minikube is using the docker driver. Ensure your docker is configured with
enough resources to satisfy the minikube start options.
```bash
$ minikube start --addons=ingress --cpus=4 --memory=8g
```

Check that we have stuff
```bash
$ kubectl get pods -A
```

### Stand up AWX using awx-operator

Clone operator, and checkout release `0.18.0`. I initially tried to use 0.28 and 0.29, but couldn't get either one to stand up.
0.18.0 _did_ work. This was an exercise to learn AWX, not how to debug its installation.
So 0.18 it is.

```bash
$ git clone git@github.com:ansible/awx-operator.git
$ cd awx-operator/
$ git checkout 0.18.0
$ export NAMESPACE=awx
$ make deploy
$ kubectl config set-context --current --namespace=${NAMESPACE}
```

Edit `awx-demo.yml`. It should be changed to:
```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
```

Apply the resource
```bash
$ kubectl apply -f awx-demo.yml
```

Tail Logs
It can take a little time to get everything up and running. May as well follow the logs for a bit.

```bash
$ kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager
```
---

#### AWX Username/ Password
The username is `admin` and password is obtained from the k8s secrets:
```bash
$ kubectl get secret awx-admin-password -o jsonpath="{.data.password}" | base64 --decode
```

### Launch Console (one-liner)
This command sets up the tunnel, gets the URL, and opens the browser
to AWX login page.
```bash
$ minikube service awx-service -n ${NAMESPACE} 
```
Output will be something like this:
```bash
|-----------|-------------|-------------|--------------|
| NAMESPACE |    NAME     | TARGET PORT |    URL       |
|-----------|-------------|-------------|--------------|
| awx       | awx-service |             |No node port  |
|-----------|-------------|-------------|--------------|
😿service awx/awx-service has no node port
🏃Starting tunnel for service awx-service.
|-----------|-------------|-------------|------------------------|
| NAMESPACE |NAME         | TARGET PORT |          URL           |
|-----------|-------------|-------------|------------------------|
| awx       | awx-service |             | http://127.0.0.1:56500 |
|-----------|-------------|-------------|------------------------|
🎉Opening service awx/awx-service in default browser...
❗Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```
----

### Launch Console (_as from original video_)
These instructions are from the original video.

Open minikube tunnel to the service:
```bash
$ minikube tunnel awx-service
```

Output will be something like this:
```bash
✅  Tunnel successfully started
📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...
```

Open a new terminal, and get the URL
```bash
$ minikube service awx-service --url -n ${NAMESPACE}
```
Output will be something like this:
```bash
😿service awx/awx-service has no node port
http://127.0.0.1:52284
❗Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```
Visit the URL reported by the command.
