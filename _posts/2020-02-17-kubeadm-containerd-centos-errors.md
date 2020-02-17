# Kubeadm Containerd Centos Errors

So if you find yourself running `kubeadm init` on centos and using Containerd as the runtime, you might hit the following preflight error:

```
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR CRI]: container runtime is not running: output: time="2020-02-17T14:45:43Z" level=fatal msg="getting status of runtime failed: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
, error: exit status 1
```

You check `systemctl status containerd` and it's running.

You check your `kubeadm init` config and see it has this block:

```
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
<<<< other stuff >>>>
nodeRegistration:
  criSocket: /var/run/containerd/containerd.sock
```

Well turns out if you `cat roles/containerd/files/config.toml` you'll see this line, which I think is the default:

```
disabled_plugins = ["cri"]
```

Change that to
```
disabled_plugins = [""]
```
and `kubeadm init` should stop complaining about the container runtime.
