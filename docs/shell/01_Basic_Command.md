# Basic Shell Command

## OS 

- Generator SSH-Key without promote

```shell
ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa <<< y
```

## Ansible

下面代码以一组 Host 生成形如 server1=ip1:port,server2=ip1:port 的结果。

Host 信息：

```shell
[node]
10.211.55.78
10.211.55.79
10.211.55.80
```

ansible对应的playbook:

```shell
- name: storage
  set_fact: host_list="{{ groups['node'] }}"
- name: combine
  set_fact: host_list="{% for item in host_list %} node{{ item.split(".")[2] }}_{{ item.split(".")[3]}}=http://{{item}}:{{ node_peer_port }} {% endfor %}"
- name: display
  set_fact: cluster_hosts={{ host_list.split() | join(",") }}
- name: debug
  debug: var=cluster_hosts
```