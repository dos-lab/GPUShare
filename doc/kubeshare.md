# KubeShare Deployment

Just use the scripts offered by its team
**NOTICE: CUDA version are recommended to be 10.0**

### Installation
```
kubectl create -f https://lsalab.cs.nthu.edu.tw/~ericyeh/KubeShare/v0.9/crd.yaml
kubectl create -f https://lsalab.cs.nthu.edu.tw/~ericyeh/KubeShare/v0.9/device-manager.yaml
kubectl create -f https://lsalab.cs.nthu.edu.tw/~ericyeh/KubeShare/v0.9/scheduler.yaml
```

### Uninstallation

```
kubectl delete -f https://lsalab.cs.nthu.edu.tw/~ericyeh/KubeShare/v0.9/crd.yaml
kubectl delete -f https://lsalab.cs.nthu.edu.tw/~ericyeh/KubeShare/v0.9/device-manager.yaml
kubectl delete -f https://lsalab.cs.nthu.edu.tw/~ericyeh/KubeShare/v0.9/scheduler.yaml
```



### Test
Put the training code in /root/gpushare/code directory on host, in the test, https://github.com/kuangliu/pytorch-cifar.git is used.
Launch 2 pods, pod1 uses 20% GPU cores and 6GB memory, pod2 uses 80% GPU cores and 6GB memory.
```yaml
apiVersion: kubeshare.nthu/v1
kind: SharePod
metadata:
  name: sharepod1
  annotations:
    "kubeshare/gpu_request": "0.2" # required if allocating GPU
    "kubeshare/gpu_limit": "0.2" # required if allocating GPU
    "kubeshare/gpu_mem": "6442450944" # required if allocating GPU # 6Gi, in bytes
    "kubeshare/GPUID": "abcde"
spec: # PodSpec
  nodeName: <node name>
  containers:
  - name: cuda
    image: pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime
    command: ["nvidia-smi", "pmon", "-d", "10"]
    volumeMounts:
      - mountPath: /root/gpushare/code
        name: code
      - mountPath: /dev/shm
        name: dshm
  volumes:
  - name: code
    hostPath:
      path: /root/gpushare/code
      type: DirectoryOrCreate
  - name: dshm
    emptyDir:
      medium: Memory
```


```yaml
apiVersion: kubeshare.nthu/v1
kind: SharePod
metadata:
  name: sharepod1
  annotations:
    "kubeshare/gpu_request": "0.8" # required if allocating GPU
    "kubeshare/gpu_limit": "0.8" # required if allocating GPU
    "kubeshare/gpu_mem": "6442450944" # required if allocating GPU # 6Gi, in bytes
    "kubeshare/GPUID": "abcde"
spec: # PodSpec
  nodeName: <node name>
  containers:
  - name: cuda
    image: pytorch/pytorch:1.6.0-cuda10.1-cudnn7-runtime
    command: ["nvidia-smi", "pmon", "-d", "10"]
    volumeMounts:
      - mountPath: /root/gpushare/code
        name: code
      - mountPath: /dev/shm
        name: dshm
  volumes:
  - name: code
    hostPath:
      path: /root/gpushare/code
      type: DirectoryOrCreate
  - name: dshm
    emptyDir:
      medium: Memory
```

After the sharepod1 is running, Enter the container to launch tasks

```
kubectl exec -it sharepod1 -- bash
$ cd /root/gpushare/code/
$ cd pytorch-cifar
$ python main.py # Run the task
```
And do the same with sharepod2

Result shows that task in sharepod2 runs ~4x faster than sharedpod1, but with no QOS garanteen.