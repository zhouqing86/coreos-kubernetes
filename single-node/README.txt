1. 下载kubectl(我的机子使用的是macos系统)
curl -O https://storage.googleapis.com/kubernetes-release/release/v1.1.2/bin/darwin/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin/kubectl

2. git clone https://github.com/coreos/coreos-kubernetes.git
cd coreos-kubernetes/single-node/
vagrant up

3. cd coreos-kubernetes/single-node/
export KUBECONFIG="${KUBECONFIG}:$(pwd)/kubeconfig"
kubectl config use-context vagrant-single
kubectl config set-cluster vagrant-single-cluster --server=https://172.17.4.99:443 --certificate-authority=${PWD}/ssl/ca.pem
kubectl config set-credentials vagrant-single-admin --certificate-authority=${PWD}/ssl/ca.pem --client-key=${PWD}/ssl/admin-key.pem --client-certificate=${PWD}/ssl/admin.pem
kubectl config set-context vagrant-single --cluster=vagrant-single-cluster --user=vagrant-single-admin
kubectl config use-context vagrant-single

kubectl的命令示例
==================================================================
kubectl run my-nginx --image=nginx --replicas=2 --port=80
kubectl get pods
kubectl get rc
kubectl stop rc my-nginx
kubectl expose rc my-nginx --port=80 --type=LoadBalancer
kubectl get services
==================================================================

如何知道kubernetes是否正常工作呢？
http://kubernetes.io/v1.1/examples/guestbook/README.html
The example consists of:
A web frontend
A redis master (for storage), and a replicated set of redis 'slaves'.
The web front end interacts with the redis master via javascript redis API calls.

1. Start Redis master
To start the redis master, use the file examples/guestbook/redis-master-controller.yaml, which describes a single pod running a redis key-value server in a container.
kubectl create -f yml/redis-master-controller.yaml
kubectl get pods will show only the pods in the default namespace. To see pods in all namespaces, run:
kubectl get pods -o wide --all-namespaces=true

You can get information about a pod, including the machine that it is running on, via kubectl describe pods/<pod_name>.
查看pod的log: kubectl logs [pod_name]

2. Fire up the redis master service
A Kubernetes service is a named load balancer that proxies traffic to one or more containers. This is done using the labels metadata that we defined in the redis-master pod above.
Services find the pods to load balance based on the pods' labels. The pod that you created in Step One has the label name=redis-master. The selector field of the service description determines which pods will receive the traffic sent to the service, and the port and targetPort information defines what port the service proxy will run at.
kubectl create -f yml/redis-master-service.yaml

Kubernetes supports two primary modes of finding a service— environment variables and DNS.
However, this is unlikely to be necessary. You can check for the DNS service in the list of the clusters' services by running kubectl --namespace=kube-system get rc, and looking for a controller prefixed kube-dns

3. Fire up the replicated slave pods
kubectl create -f redis-slave-controller.yaml

4. Create the redis slave service
This time the selector for the service is name=redis-slave, because that identifies the pods running redis slaves. It may also be helpful to set labels on your service itself as we've done here to make it easy to locate them with the kubectl get services -l "label=value" command.
kubectl create -f yml/redis-slave-service.yaml

5. Create the frontend replicated pods
kubectl create -f yml/frontend-controller.yaml

6. Set up the guestbook frontend service.
kubectl create -f yml/frontend-service.yaml
kubectl get services

kubectl delete -f yml/frontend-service.yaml

7. Accessing the guestbook site externally
You'll want to set up your guestbook service so that it can be accessed from outside of the internal Kubernetes network. Above, we introduced one way to do that, using the type: LoadBalancer spec.
More generally, Kubernetes supports two ways of exposing a service onto an external IP address: NodePorts and LoadBalancers

If the LoadBalancer specification is used, it can take a short period for an external IP to show up in kubectl get services output, but you should shortly see it listed as well.

stop和delete service
kubectl stop rc -l "name in (redis-master, redis-slave, frontend)"
kubectl delete service -l "name in (redis-master, redis-slave, frontend)"