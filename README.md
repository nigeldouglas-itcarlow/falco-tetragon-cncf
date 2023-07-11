# falco-tetragon-cncf
Running Falco &amp; Tetragon together to achieve cloud-native detection &amp; response (CDR)

```
eksctl create cluster nigel-eks-cluster --node-type t3.xlarge --nodes=1 --nodes-min=0 --nodes-max=3 --max-pods-per-node 58
```
![Screenshot 2023-07-11 at 13 16 55](https://github.com/nigeldouglas-itcarlow/falco-tetragon-cncf/assets/126002808/8a4876ac-6d1d-4131-918e-19f64f5dc7e1)

## Install Falco
```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
helm install falco falcosecurity/falco --namespace falco --create-namespace
kubectl get pods -n falco -o wide -w
```

![Screenshot 2023-07-11 at 13 21 57](https://github.com/nigeldouglas-itcarlow/falco-tetragon-cncf/assets/126002808/33ea3e03-f041-4c48-9055-17f5256612c2)

Get the logs from the pod falco-rq4gs
```
kubectl logs falco-rq4gs -n falco
```
