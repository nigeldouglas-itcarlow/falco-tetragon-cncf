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

You can use the ```kubectl get pods``` command along with some additional filters and the ```--output=jsonpath``` option to retrieve the pod name dynamically and then pass it to the ```kubectl logs``` command.
```
POD_NAME=$(kubectl get pods -n falco --field-selector=status.phase=Running -o=jsonpath='{.items[0].metadata.name}')
kubectl logs $POD_NAME -n falco
```

This command fetches the name of the first running pod in the ```falco``` namespace and assigns it to the ```POD_NAME``` variable. <br/>
Then, it uses the retrieved pod name to fetch the logs of that pod using ```kubectl logs```.

![Screenshot 2023-07-11 at 13 25 30](https://github.com/nigeldouglas-itcarlow/falco-tetragon-cncf/assets/126002808/b64d4c0c-dfbf-4664-8096-121965724dd1)

alternatively, you can run:
```
kubectl logs -n falco -l app.kubernetes.io/instance=falco
```

![Screenshot 2023-07-11 at 13 49 16](https://github.com/nigeldouglas-itcarlow/falco-tetragon-cncf/assets/126002808/72aa9201-8f94-4ecb-a801-5030c36148a4)


## Install Tetragon
```
helm repo add cilium https://helm.cilium.io
helm repo update
helm install tetragon cilium/tetragon -n kube-system
kubectl rollout status -n kube-system ds/tetragon -w
```

![Screenshot 2023-07-11 at 13 31 01](https://github.com/nigeldouglas-itcarlow/falco-tetragon-cncf/assets/126002808/72c61a0c-a3d6-45d5-9ace-1b19929666bc)


