

Ravindra Kumar






Ravindra Kumar
Ravindra Kumar
May 20

·
5 min read
·

Listen







Install Kubernetes using Rancher
Environment

Setup Ansible connection to all machines:
==================
Set up passwordless connection to all machines:
==================
ssh-copy-id root@172.16.199.115
ssh-copy-id root@172.16.199.116
ssh-copy-id root@172.16.199.117
==================
ansible inventory host file:
==================
master ansible_host=”172.16.199.115" ansible_user=root
node1 ansible_host=”172.16.199.116" ansible_user=root
node2 ansible_host=”172.16.199.117" ansible_user=root
=============
ansible.cfg file: (sample ansible.cfg)
=============
[defaults]
inventory = ./hosts
host_key_checking = False
stdout_callback = yaml
bin_ansible_callbacks=True
[inventory]
[privilege_escalation]
become=True
become_method=sudo
become_user=root
[paramiko_connection]
[ssh_connection]
[persistent_connection]
[accelerate]
[selinux]
[colors]
highlight = white
verbose = blue
warn = bright purple
error = red
debug = dark gray
deprecate = purple
skip = cyan
unreachable = red
ok = green
changed = yellow
diff_add = green
diff_remove = red
diff_lines = cyan
================
Implement ssh root login
================
ansible -v all -m shell -a “echo ‘PermitRootLogin yes’ >> /etc/ssh/sshd_config”
ansible -v all -m shell -a “systemctl restart sshd”
=============
Copy ssh keys to all nodes
=============
ssh-copy-id root@172.16.199.115
ssh-copy-id root@172.16.199.116
ssh-copy-id root@172.16.199.117
=============
check ansible connectivity
============
ravindrakumar@Ravindras-iMac-Pro rancher % ansible all -m ping

PLAY [Ansible Ad-Hoc] *****************************************************************************************

TASK [ping] ***************************************************************************************************
ok: [node1]
ok: [master]
ok: [node2]

PLAY RECAP ****************************************************************************************************
master : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
node1 : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
node2 : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0

Setup pre-requisites
==========
Install Docker
==========
ansible -v all -m script -a “./docker.sh”
docker.sh file:
apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository “deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable”
apt update && apt install -y docker-ce=5:19.03.10~3–0~ubuntu-focal containerd.io
==========
Setup Sysctl (sysctl.sh) and disable swap
==========
ansible -v all -m script -a “./sysctl.sh”
sysctl.sh file:
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl — system
ansible -v all -m shell -a “swapoff -a; sed -i ‘/swap/d’ /etc/fstab”

Install kubernetes
==========
Create cluster.yml
==========
# If you intended to deploy Kubernetes in an air-gapped environment,
# please consult the documentation on how to configure custom RKE images.
nodes:
- address: “172.16.199.115”
port: “22”
internal_address: “172.16.199.115”
role:
— controlplane
— worker
— etcd
hostname_override: 172.16.199.115
user: root
docker_socket: /var/run/docker.sock
ssh_key: “”
ssh_key_path: ~/.ssh/id_rsa
ssh_cert: “”
ssh_cert_path: “”
labels: {}
taints: []
- address: “172.16.199.116”
port: “22”
internal_address: 172.16.199.116
role:
— controlplane
— worker
— etcd
hostname_override: 172.16.199.116
user: root
docker_socket: /var/run/docker.sock
ssh_key: “”
ssh_key_path: ~/.ssh/id_rsa
ssh_cert: “”
ssh_cert_path: “”
labels: {}
taints: []
- address: “172.16.199.117”
port: “22”
internal_address: “172.16.199.117”
role:
— controlplane
— worker
— etcd
hostname_override: “172.16.199.117”
user: root
docker_socket: /var/run/docker.sock
ssh_key: “”
ssh_key_path: ~/.ssh/id_rsa
ssh_cert: “”
ssh_cert_path: “”
labels: {}
taints: []
services:
etcd:
image: “”
extra_args: {}
extra_binds: []
extra_env: []
win_extra_args: {}
win_extra_binds: []
win_extra_env: []
external_urls: []
ca_cert: “”
cert: “”
key: “”
path: “”
uid: 0
gid: 0
snapshot: null
retention: “”
creation: “”
backup_config: null
kube-api:
image: “”
extra_args: {}
extra_binds: []
extra_env: []
win_extra_args: {}
win_extra_binds: []
win_extra_env: []
service_cluster_ip_range: 10.43.0.0/16
service_node_port_range: “”
pod_security_policy: false
always_pull_images: false
secrets_encryption_config: null
audit_log: null
admission_configuration: null
event_rate_limit: null
kube-controller:
image: “”
extra_args: {}
extra_binds: []
extra_env: []
win_extra_args: {}
win_extra_binds: []
win_extra_env: []
cluster_cidr: 10.42.0.0/16
service_cluster_ip_range: 10.43.0.0/16
scheduler:
image: “”
extra_args: {}
extra_binds: []
extra_env: []
win_extra_args: {}
win_extra_binds: []
win_extra_env: []
kubelet:
image: “”
extra_args: {}
extra_binds: []
extra_env: []
win_extra_args: {}
win_extra_binds: []
win_extra_env: []
cluster_domain: cluster.local
infra_container_image: “”
cluster_dns_server: 10.43.0.10
fail_swap_on: false
generate_serving_certificate: false
kubeproxy:
image: “”
extra_args: {}
extra_binds: []
extra_env: []
win_extra_args: {}
win_extra_binds: []
win_extra_env: []
network:
plugin: calico
options: {}
mtu: 0
node_selector: {}
update_strategy: null
tolerations: []
authentication:
strategy: x509
sans: []
webhook: null
addons: “”
addons_include: []
system_images:
etcd: rancher/mirrored-coreos-etcd:v3.5.0
alpine: rancher/rke-tools:v0.1.80
nginx_proxy: rancher/rke-tools:v0.1.80
cert_downloader: rancher/rke-tools:v0.1.80
kubernetes_services_sidecar: rancher/rke-tools:v0.1.80
kubedns: rancher/mirrored-k8s-dns-kube-dns:1.17.4
dnsmasq: rancher/mirrored-k8s-dns-dnsmasq-nanny:1.17.4
kubedns_sidecar: rancher/mirrored-k8s-dns-sidecar:1.17.4
kubedns_autoscaler: rancher/mirrored-cluster-proportional-autoscaler:1.8.3
coredns: rancher/mirrored-coredns-coredns:1.8.6
coredns_autoscaler: rancher/mirrored-cluster-proportional-autoscaler:1.8.5
nodelocal: rancher/mirrored-k8s-dns-node-cache:1.21.1
kubernetes: rancher/hyperkube:v1.22.7-rancher1
flannel: rancher/mirrored-coreos-flannel:v0.15.1
flannel_cni: rancher/flannel-cni:v0.3.0-rancher6
calico_node: rancher/mirrored-calico-node:v3.21.1
calico_cni: rancher/mirrored-calico-cni:v3.21.1
calico_controllers: rancher/mirrored-calico-kube-controllers:v3.21.1
calico_ctl: rancher/mirrored-calico-ctl:v3.21.1
calico_flexvol: rancher/mirrored-calico-pod2daemon-flexvol:v3.21.1
canal_node: rancher/mirrored-calico-node:v3.21.1
canal_cni: rancher/mirrored-calico-cni:v3.21.1
canal_controllers: rancher/mirrored-calico-kube-controllers:v3.21.1
canal_flannel: rancher/mirrored-flannelcni-flannel:v0.17.0
canal_flexvol: rancher/mirrored-calico-pod2daemon-flexvol:v3.21.1
weave_node: weaveworks/weave-kube:2.8.1
weave_cni: weaveworks/weave-npc:2.8.1
pod_infra_container: rancher/mirrored-pause:3.6
ingress: rancher/nginx-ingress-controller:nginx-1.1.0-rancher1
ingress_backend: rancher/mirrored-nginx-ingress-controller-defaultbackend:1.5-rancher1
ingress_webhook: rancher/mirrored-ingress-nginx-kube-webhook-certgen:v1.1.1
metrics_server: rancher/mirrored-metrics-server:v0.5.1
windows_pod_infra_container: rancher/mirrored-pause:3.6
aci_cni_deploy_container: noiro/cnideploy:5.1.1.0.1ae238a
aci_host_container: noiro/aci-containers-host:5.1.1.0.1ae238a
aci_opflex_container: noiro/opflex:5.1.1.0.1ae238a
aci_mcast_container: noiro/opflex:5.1.1.0.1ae238a
aci_ovs_container: noiro/openvswitch:5.1.1.0.1ae238a
aci_controller_container: noiro/aci-containers-controller:5.1.1.0.1ae238a
aci_gbp_server_container: noiro/gbp-server:5.1.1.0.1ae238a
aci_opflex_server_container: noiro/opflex-server:5.1.1.0.1ae238a
ssh_key_path: ~/.ssh/id_rsa
ssh_cert_path: “”
ssh_agent_auth: false
authorization:
mode: rbac
options: {}
ignore_docker_version: null
enable_cri_dockerd: null
kubernetes_version: “”
private_registries: []
ingress:
provider: “”
options: {}
node_selector: {}
extra_args: {}
dns_policy: “”
extra_envs: []
extra_volumes: []
extra_volume_mounts: []
update_strategy: null
http_port: 0
https_port: 0
network_mode: “”
tolerations: []
default_backend: null
default_http_backend_priority_class_name: “”
nginx_ingress_controller_priority_class_name: “”
default_ingress_class: null
cluster_name: “”
cloud_provider:
name: “”
prefix_path: “”
win_prefix_path: “”
addon_job_timeout: 0
bastion_host:
address: “”
port: “”
user: “”
ssh_key: “”
ssh_key_path: “”
ssh_cert: “”
ssh_cert_path: “”
ignore_proxy_env_vars: false
monitoring:
provider: “”
options: {}
node_selector: {}
update_strategy: null
replicas: null
tolerations: []
metrics_server_priority_class_name: “”
restore:
restore: false
snapshot_name: “”
rotate_encryption_key: false
dns: null
==========
Run the rancher script
==========

ravindrakumar@Ravindras-iMac-Pro rancher % rke up
INFO[0000] Running RKE version: v1.3.8
INFO[0000] Initiating Kubernetes cluster
INFO[0000] [certificates] GenerateServingCertificate is disabled, checking if there are unused kubelet certificates
— truncated for brevity —
INFO[0255] [addons] Executing deploy job rke-ingress-controller
INFO[0265] [ingress] removing default backend service and deployment if they exist
INFO[0265] [ingress] ingress controller nginx deployed successfully
INFO[0265] [addons] Setting up user addons
INFO[0265] [addons] no user addons defined
INFO[0265] Finished building Kubernetes cluster successfully

Copy the kube config file to home directory .kube location

% mkdir ~/.kube
% cp kube_config_cluster.yml ~/.kube/config

=========
check cluster status
=========
% kubectl cluster-info
Kubernetes control plane is running at https://172.16.199.117:6443
CoreDNS is running at https://172.16.199.117:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use ‘kubectl cluster-info dump’.

% kubectl get nodes
NAME STATUS ROLES AGE VERSION
172.16.199.115 Ready controlplane,etcd,worker 30m v1.22.7
172.16.199.116 Ready controlplane,etcd,worker 30m v1.22.7
172.16.199.117 Ready controlplane,etcd,worker 30m v1.22.7




