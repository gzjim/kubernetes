## Glusterfs

[Glusterfs](http://www.gluster.org) is an open source scale-out filesystem. These examples provide information about how to allow containers use Glusterfs volumes.

The example assumes that you have already set up a Glusterfs server cluster and the Glusterfs client package is installed on all Kubernetes nodes.

### Prerequisites

Set up Glusterfs server cluster; install Glusterfs client package on the Kubernetes nodes. ([Guide](https://www.howtoforge.com/high-availability-storage-with-glusterfs-3.2.x-on-debian-wheezy-automatic-file-replication-mirror-across-two-storage-servers))

### Create endpoints
Here is a snippet of [glusterfs-endpoints.json](glusterfs-endpoints.json),

```
      "addresses": [
        {
          "IP": "10.240.106.152"
        }
      ],
      "ports": [
        {
          "port": 1,
          "protocol": "TCP"
        }
      ]

```
The "IP" field should be filled with the address of a node in the Glusterfs server cluster. In this example, it is fine to give any valid value (from 1 to 65535) to the "port" field. 

Create the endpoints,
```shell
$ kubectl create -f examples/glusterfs/glusterfs-endpoints.json
```

You can verify that the endpoints are successfully created by running
```shell
$ kubect get endpoints
NAME                ENDPOINTS
glusterfs-cluster   10.240.106.152:1,10.240.79.157:1
```

### Create a POD

The following *volume* spec in [glusterfs-pod.json](glusterfs-pod.json) illustrates a sample configuration.

```js
{
     "name": "glusterfsvol",
     "glusterfs": {
        "endpoints": "glusterfs-cluster",
        "path": "kube_vol",
        "readOnly": true
    }
}
```

The parameters are explained as the followings. 

- **endpoints** is endpoints name that represents a Gluster cluster configuration. *kubelet* is optimized to avoid mount storm, it will randomly pick one from the endpoints to mount. If this host is unresponsive, the next Gluster host in the endpoints is automatically selected. 
- **path** is the Glusterfs volume name. 
- **readOnly** is the boolean that sets the mountpoint readOnly or readWrite. 

Create a pod that has a container using Glusterfs volume,
```shell
$ kubectl create -f examples/glusterfs/glusterfs-pod.json
```
You can verify that the pod is running:

```shell
$ kubectl get pods
POD         IP            CONTAINER(S)   IMAGE(S)              HOST                                  LABELS    STATUS    CREATED          MESSAGE
glusterfs   10.244.2.13                                        kubernetes-minion-151f/23.236.54.97   <none>    Running   About a minute   
                          glusterfs      kubernetes/pause                                                      Running   About a minute   

```

You may ssh to the host and run 'mount' to see if the Glusterfs volume is mounted,
```shell
$ mount | grep kube_vol
10.240.106.152:kube_vol on /var/lib/kubelet/pods/f164a571-fa68-11e4-ad5c-42010af019b7/volumes/kubernetes.io~glusterfs/glusterfsvol type fuse.glusterfs (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,max_read=131072)
```

You may also run `docker ps` on the host to see the actual container.


[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/examples/glusterfs/README.md?pixel)]()


[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/release-0.19.0/examples/glusterfs/README.md?pixel)]()
