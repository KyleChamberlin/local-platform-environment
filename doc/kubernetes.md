## 2. Local Kubernetes install/setup (with userspace virtualization, on macos) 

There are many different local kubernetes options. Things to consider when choosing among various options:  
- Metrics APIs: Can i run all the standard metrics apis? (metrics-server, kube-state-metrics)
- Mesh: Does it support use of Istio?
- Ingress: Is there a means of routing local traffic into the Istio ingress controller using the same url pattern as the live service?
- Deployment: Can I use the exact same deployment mechanism (helm, argocd, kustomize) with local environment parameters to deploy containers?

### minikube

Virtualization running in Userspace will consistently outperform full virtualization, all else being equal. RAM is a potentially critical requirement.  

• [minikube](https://minikube.sigs.k8s.io)

• configure minikube settings and start  

• Note, m1 (apple silicon) macs do not support hyperkit. A list of alternatives for vm-driver is here: https://minikube.sigs.k8s.io/docs/drivers/

Typical settings for a MacBook Pro with 16Gb ram, but adjust to fit your machine.  

```bash
$ minikube config set vm-driver hyperkit
$ minikube config set memory 12288
$ minikube config set cpus 4
```

Start minikube with the following options:  
```bash
$ minikube start \
--container-runtime=containerd \
--insecure-registry "10.0.0.0/24" \
--addons metrics-server enable
```

To support rapid, local image build iterations, launch a local registry:  
```bash
$ nerdctl run -d --rm -it --name=registry-fwd --network=host alpine ash -c \"apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:$(minikube ip):5000\"
```

Run the local loadbalancer to route traffic:  
```bash
$ minikube tunnel &
```

You can use the invoke helper script to start minikube wiith the above configuration:  
```bash
$ inv k8s.start
$ inv k8s.registry  # launnches registry addon and port re-redirect container
```

To remove the registry network forwarder use and delete the minikube vm:
```
$ docker stop registry-fwd
$ minikube delete
```

### Local k8s security profile

To see how the default local development configuration compares to EKS using the CIS benchmark:  
```bash
$ inv k8s.bench
```

[Return](../README.md)
