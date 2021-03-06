# Kubeadm Ansible Playbook Tested only on CentOS7

Build a Kubernetes cluster using Ansible with kubeadm. The goal is easily install a Kubernetes cluster on machines running:

  - CentOS 7

System requirements:

  - Deployment environment must have Ansible `2.4.0+`
  - Master and nodes must have passwordless SSH access

# Usage

Add the system information gathered above into a file called `hosts.ini`. For example:
```
[master]
master ansible_ssh_host=<Master Server IP> ansible_ssh_user=<user name> ansible_ssh_private_key_file=<private key file path for master  IP>


[node]
node1 ansible_ssh_host=<Node1 Server IP> ansible_ssh_user=<user name> ansible_ssh_private_key_file=<private key file path for Node 1 IP>
node2 ansible_ssh_host=<Node2 Server IP> ansible_ssh_user=<user name> ansible_ssh_private_key_file=<private key file path for Node 2 IP>

[kube-cluster:children]
master
node
```

Before continuing, edit `group_vars/all.yml` to your specified configuration.

For example, I choose to run `flannel` instead of calico, and thus:

```yaml
# Network implementation('flannel', 'calico')
network: flannel
```

**Note:** Depending on your setup, you may need to modify `cni_opts` to an available network interface. By default, `kubeadm-ansible` uses `eth1`. Your default interface may be `eth0`.

After going through the setup, run the `site.yaml` playbook:

```sh
$ ansible-playbook site.yaml

Download `/etc/kubernetes/admin.conf` file to `$HOME/admin.conf`.

```sh
$ scp k8s@k8s-master:/etc/kubernetes/admin.conf .
```

Verify cluster is fully running using kubectl:

```sh

$ export KUBECONFIG=~/admin.conf
$ kubectl get node
NAME      STATUS    AGE       VERSION
master1   Ready     22m       v1.15.1
node1     Ready     20m       v1.15.1
node2     Ready     20m       v1.15.1

$ kubectl get po -n kube-system
NAME                                    READY     STATUS    RESTARTS   AGE
etcd-master1                            1/1       Running   0          23m
...
```

# Resetting the environment

Finally, reset all kubeadm installed state using `reset-site.yaml` playbook:

```sh
$ ansible-playbook reset-site.yaml
```

# Additional features
These are features that you could want to install to make your life easier.

Enable/disable these features in `group_vars/all.yml` (all disabled by default):
```
# Additional feature to install
additional_features:
  helm: true
  healthcheck: true
  elasticsearch: true
  prometheus: true
```

## Helm
This will install helm in your cluster (https://helm.sh/) so you can deploy charts.

## Healthcheck
This will install k8s-healthcheck (https://github.com/emrekenci/k8s-healthcheck), a small application to report cluster status.

# Prometheus-Grafana
- access Grafana via http://{IP}:3000/
- Enable the app at http://{IP}:3000/plugins/grafana-kubernetes-app/edit
- Add a data source of prometheus type: http://{IP}:3000/datasources/new

# ElasticSearch-Kibana-Fluentd
- run `kubectl port-forward kibana 5601:5601 --namespace=kube-logging
