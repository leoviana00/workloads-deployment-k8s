## Kubespray config files
1. Configuração do `all.yaml`: Dns, ip e porta
```yaml
## Directory where the binaries will be installed
bin_dir: /usr/local/bin

## External LB example config
apiserver_loadbalancer_domain_name: "lb.lab.k8s.io"
loadbalancer_apiserver:
  address: 10.0.2.2
  port: 6443

## Local loadbalancer should use this port
## And must be set port 6443
loadbalancer_apiserver_port: 6443

## If loadbalancer_apiserver_healthcheck_port variable defined, enables proxy liveness check for nginx.
loadbalancer_apiserver_healthcheck_port: 8081

## Upstream dns servers
nameservers:
  - 8.8.8.8
  - 8.8.4.4

no_proxy_exclude_workers: false
```

2. Configuração `etcd.yaml`
```yaml
## Directory where etcd data stored
etcd_data_dir: /var/lib/etcd

etcd_deployment_type: host

```

3. Configuração `k8s-cluster.yaml`
```yaml
kube_config_dir: /etc/kubernetes
kube_script_dir: "{{ bin_dir }}/kubernetes-scripts"
kube_manifest_dir: "{{ kube_config_dir }}/manifests"

# This is where all the cert scripts and certs will be located
kube_cert_dir: "{{ kube_config_dir }}/ssl"

# This is where all of the bearer tokens will be stored
kube_token_dir: "{{ kube_config_dir }}/tokens"

kube_api_anonymous_auth: true

kube_version: v1.23.5

# Note: ensure that you've enough disk space (about 1G)
local_release_dir: "/tmp/releases"

 Random shifts for retrying failed ops like pushing/downloading
retry_stagger: 5

kube_cert_group: kube-cert

# Cluster Loglevel configuration
kube_log_level: 2

# Directory where credentials will be stored
credentials_dir: "{{ inventory_dir }}/credentials"

kube_network_plugin: calico
kube_network_plugin_multus: false
kube_service_addresses: 10.233.0.0/18
kube_pods_subnet: 10.233.64.0/18
kube_network_node_prefix: 24
# Configure Dual Stack networking (i.e. both IPv4 and IPv6)
enable_dual_stack_networks: false
kube_service_addresses_ipv6: fd85:ee78:d8a6:8607::1000/116
kube_pods_subnet_ipv6: fd85:ee78:d8a6:8607::1:0000/112
kube_network_node_prefix_ipv6: 120

# The port the API Server will be listening on.
kube_apiserver_ip: "{{ kube_service_addresses|ipaddr('net')|ipaddr(1)|ipaddr('address') }}"
kube_apiserver_port: 6443  # (https)

kube_apiserver_insecure_port: 0  # (disabled)

# Kube-proxy proxyMode configuration.
# Can be ipvs, iptables
kube_proxy_mode: ipvs
kube_proxy_strict_arp: false

kube_proxy_nodeport_addresses: >-
  {%- if kube_proxy_nodeport_addresses_cidr is defined -%}
  [{{ kube_proxy_nodeport_addresses_cidr }}]
  {%- else -%}
  []
  {%- endif -%}

kube_encrypt_secret_data: false
cluster_name: cluster.local
ndots: 2
dns_mode: coredns
enable_nodelocaldns: true
enable_nodelocaldns_secondary: false
nodelocaldns_ip: 169.254.25.10
nodelocaldns_health_port: 9256
nodelocaldns_bind_metrics_host_ip: false
nodelocaldns_secondary_skew_seconds: 5


enable_coredns_k8s_external: false
coredns_k8s_external_zone: k8s_external.local
# Enable endpoint_pod_names option for kubernetes plugin
enable_coredns_k8s_endpoint_pod_names: false

# Can be docker_dns, host_resolvconf or none
resolvconf_mode: host_resolvconf
# Deploy netchecker app to verify DNS resolve as an HTTP service
deploy_netchecker: false
# Ip address of the kubernetes skydns service
skydns_server: "{{ kube_service_addresses|ipaddr('net')|ipaddr(3)|ipaddr('address') }}"
skydns_server_secondary: "{{ kube_service_addresses|ipaddr('net')|ipaddr(4)|ipaddr('address') }}"
dns_domain: "{{ cluster_name }}"

## Container runtime
## docker for docker, crio for cri-o and containerd for containerd.
container_manager: containerd

# Additional container runtimes
kata_containers_enabled: false

kubeadm_certificate_key: "{{ lookup('password', credentials_dir + '/kubeadm_certificate_key.creds length=64 chars=hexdigits') | lower }}"

# K8s image pull policy (imagePullPolicy)
k8s_image_pull_policy: IfNotPresent

# audit log for kubernetes
kubernetes_audit: false

# dynamic kubelet configuration
dynamic_kubelet_configuration: false

# define kubelet config dir for dynamic kubelet
# kubelet_config_dir:
default_kubelet_config_dir: "{{ kube_config_dir }}/dynamic_kubelet_dir"
dynamic_kubelet_configuration_dir: "{{ kubelet_config_dir | default(default_kubelet_config_dir) }}"

# pod security policy (RBAC must be enabled either by having 'RBAC' in authorization_modes or kubeadm enabled)
podsecuritypolicy_enabled: false

volume_cross_zone_attachment: false
persistent_volumes_enabled: false

## Amount of time to retain events. (default 1h0m0s)
event_ttl_duration: "1h0m0s"
##  Force regeneration of kubernetes control plane certificates without the need of bumping the cluster version
force_certificate_regeneration: false
```

4. Configuração `k8s-net-calico.yaml`
```yaml
calico_pool_cidr: 10.233.64.0/18
```

5. `hosts.yaml`
```yaml
all:
  vars: 
    cluster_id: "1.0.0.1"
    ansible_ssh_private_key_file: ../keys/kubespray

  hosts:
    # -------- ETCD ---------
    k8s-lab-etcd-1:
      ansible_host: 192.168.50.31
      ip: 192.168.50.31
      access_ip: 192.168.50.31
    # k8s-lab-etcd-2:
    #   ansible_host: 192.168.50.32
    #   ip: 192.168.50.32
    #   access_ip: 192.168.50.32
    # -------- MASTER ---------
    k8s-lab-master-1:
      ansible_host: 192.168.50.11
      ip: 192.168.50.11
      access_ip: 192.168.50.11
    # k8s-lab-master-2:
    #   ansible_host: 192.168.50.12
    #   ip: 192.168.50.12
    #   access_ip: 192.168.50.12
    # -------- NODE ---------
    k8s-lab-node-1:
      ansible_host: 192.168.50.21
      ip: 192.168.50.21
      access_ip: 192.168.50.21
    k8s-lab-node-2:
      ansible_host: 192.168.50.22
      ip: 192.168.50.22
      access_ip: 192.168.50.22
    # -------- CALICO ---------
    k8s-lab-rr-1:
      ansible_host: 192.168.50.41
      ip: 192.168.50.41
      access_ip: 192.168.50.41
    # k8s-lab-rr-2:
    #   ansible_host: 192.168.50.42
    #   ip: 192.168.50.42
    #   access_ip: 192.168.50.42

  children:
    kube-master:
      hosts:
        k8s-lab-master-1:
        # k8s-lab-master-2:
    kube-node:
      hosts:
        k8s-lab-master-1:
        # k8s-lab-master-2:
        k8s-lab-node-1:
        k8s-lab-node-2:
    etcd:
      hosts:
        k8s-lab-etcd-1:
        # k8s-lab-etcd-2:
    calico-rr:
      hosts: 
        k8s-lab-rr-1:
        # k8s-lab-rr-2:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
        calico-rr:
```

## Configurar KUBCTL

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```