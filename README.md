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

## Terminal Shell in Container

Create a workload with ```privileged=true``` on ```Terminal 1```:
```
kubectl apply -f https://raw.githubusercontent.com/nigeldouglas-itcarlow/Tetragon-Lab/main/privileged-pod.yaml
```

Open an activity tail for Tetragon (```Terminal 2```):
```
kubectl logs -n kube-system -l app.kubernetes.io/name=tetragon -c export-stdout -f | tetra getevents -o compact --namespace default --pod test-pod-1
```

Open an event output for Falco (```Terminal 3```):
```
kubectl logs -n falco -l app.kubernetes.io/instance=falco -w
```

Shell into the workload on ```Terminal 1```:
```
kubectl exec -it nigel-app -- bash
```

```Tetragon``` in this case is providing a ```Live Tail``` of all process-related activity in real-time. <br/>
```Falco``` is providing triggering ```Realtime Alerts``` based on the same system call activity observed by Tetragon:

![Screenshot 2023-07-12 at 14 45 23](https://github.com/nigeldouglas-itcarlow/falco-tetragon-cncf/assets/126002808/941357c0-704c-4e25-b5b1-c09544fc8116)


## Cryptomining Binary Detection
```
wget https://raw.githubusercontent.com/nigeldouglas-itcarlow/falco-tetragon-cncf/main/config/custom-rules.yaml
```

```
cat custom-rules.yaml
```

```
helm install falco -f custom-rules.yaml falcosecurity/falco
```
And we will see in our logs something like:
```
Mon Jan 30 10:56:26 2023: Loading rules from file /etc/falco/rules.d/rules-mining.yaml:
```
