# Mutual TLS with Linkerd

> Visit Linkerd documentation if you need a refresher about [service mesh](https://linkerd.io/what-is-a-service-mesh/)
>
> Basic definitions are provided on diagrams below
> This scenario is part of a [blog about mTLS with linkerd]()

## How does it work?

Once installed on the cluster, linkerd control plane will inject sidecars to Kubernetes system pods. From there we can inject sidecars to the pods and create a service mesh.

![Linkerd mTLS](http://www.plantuml.com/plantuml/proxy?cache=yes&src=https://raw.githubusercontent.com/Piotr1215/dca-prep-kit/master/diagrams/linkerd-mtls-sequence.puml&fmt=png)

## What Problem does it solve?

Service meshes add observability, security, and reliability features to “cloud native” applications by transparently inserting this functionality at the platform layer rather than the application layer.

- **Platform Level Metrics**: without changing configuration or source code, track low level metrics
- **Mutual TLS - mLTS**: add encryption and certificates based identity to cluster workloads
- **Improved Resiliency**: latency aware load balancing, retires, timeouts and advanced deployment patterns
- **Authorization Policy**: enforce traffic rules on services level

In next steps of this tutorial we will focus on setting up and utilizing mutualTLS with Linkerd

☕ background script will perform following tasks:

> IMPORTANT. If any of the steps fails or you see CrashLoopBackOff in the kubectl command results, please restart the Katacoda environment.

- start a 2-node Kubernetes cluster
- prepare katacoda environment
- setup required environmental variables
- install linkerd CLI
- display information about the cluster

> Once all nodes are ready, we can see cluster health information.

Check what pods are deployed in *kube-system* namespace:

`kubectl get pods -n kube-system`

Let's also make sure that the linkerd CLI was succesfully installed.

`linkerd version`

And check if the cluster is ready for the control plane installation

`linkerd check --pre`

Once all the checks are green, proceed to the next step 👟
📎 setup linkerd control plane

➡ install linkerd

```bash
linkerd install | kubectl apply -f -
```

➡ check linkerd installation

`linkerd check`

> be patient, this can take a while ⌛
💻 Setup Frontent

> Kuard is a demo K8s application from the book “Kubernetes Up and Running”

➡ create Kuard deployment

`kubectl create deployment --image=gcr.io/kuar-demo/kuard-amd64:blue kuard`

➡ wait for the pod to come up

`kubectl wait deployment kuard --for=condition=Available --timeout=1m`

➡ forward traffic to the pod

`kubectl port-forward deploy/kuard 8080:8080 --address 0.0.0.0 &`

Naviagate to [the kuard page](https://[[HOST_SUBDOMAIN]]-8080-[[KATACODA_HOST]].environments.katacoda.com/), or open it from the tabs on top of the terminal window.

![kuard-app](assets/kuard-app.png)
💉 inject linkerd sidecar

> Before injecting the linkerd sidecar, let's see how many containers run inside kuard pod. There should be only kuard container running.

`kubectl get pods -l app=kuard -o jsonpath='{.items[*].spec.containers[*].name}{"\n"}'`

➡ inject linkerd sidecar into kuard pod

```bash
kubectl get deploy kuard -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

➡ make sure linkerd sidecar applied correctly

`linkerd -n default check --proxy`

➡ we should see the linkerd sidecar running in the pod

`kubectl get pods -l app=kuard -o jsonpath='{.items[*].spec.containers[*].name}{"\n"}'`
🔭 Observability

➡ first we need to install observability extension

`helm repo add linkerd https://helm.linkerd.io/stable`

```bash
helm install linkerd-viz \
  --set dashboard.enforcedHostRegexp=.* \
  linkerd/linkerd-viz
```

➡ make sure the installation succedded

`linkerd check`

➡ launch the dashboard in background

```bash
linkerd viz dashboard --address 0.0.0.0 &
```

➡ visit the page or access from the tabs on top of terminal

[Linkerd Dashboard](https://[[HOST_SUBDOMAIN]]-50750-[[KATACODA_HOST]].environments.katacoda.com/)

➡ check if mTLS works between system pods and kuard pod

![kuard-deployment](assets/kuard-deployment.png)
👮 Mutual TLS

> All pods with the injected linkerd sidecar communicate by default with encrypted and authenticated traffic

➡ restart pod to enable tap configuration

```bash
kubectl rollout restart deployment kuard
```

➡ explore dashboard to see connectivity details

![secure-connection](assets/secure-connection.png)
# And we are done

Thank you for taking the time to go through my tutorial, I hope you were able to gain practical understanding of how Kubernetes deployments work. Please consider opening pull request or an issue in my [katacoda scenarios repository](https://github.com/Piotr1215/katacoda-scenarios) if you find errors or have improvements suggestions. Finally if you want to chat I'm active on [twitter](https://twitter.com/piotr1215)

Check out my other blogs on [medium](https://piotrzan.medium.com/) or contribute to some of my [open source projects](https://github.com/Piotr1215)

## Credits

While creating this tutorial I have used a lot of open source ideas and repositories, to name a few:

- [katacoda scenarios](https://github.com/amblina/katacoda-scenarios) by [amblina](https://github.com/amblina)
- [katacoda networking introduction](https://www.katacoda.com/courses/kubernetes/networking-introduction) by [katacoda](https://github.com/katacoda)
- [Kubernetes lab setup](https://github.com/loodse/kubernetes-lab) by [loodse](https://github.com/loodse)
- [The All-in-One Playground for Kubernetes](https://github.com/morningspace/lab-k8s-playground) by [morningspace](https://github.com/morningspace)

