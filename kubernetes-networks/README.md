sudo minikube start --driver=podman --container-runtime=containerd --vm-driver=kvm
minikube start --vm-driver=kvm


minikube start --extra-config kube-proxy.mode=ipvs --extra-config kube-proxy.ipvs.strictARP=true 


modprobe -a ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh nf_conntrack_ipv4
vi /etc/sysctl.conf 
net.ipv4.vs.conntrack = 1
sudo sysctl -p

minikube start  --container-runtime=containerd --vm-driver=kvm --addons=metallb --extra-config kube-proxy.mode=ipvs --extra-config kube-proxy.ipvs.strictARP=true 
kubectl edit configmap -n kube-system kube-proxy
kk delete po -n kube-system kube-proxy-qh4w8
kk get po -n kube-system


cat web-pod.yaml
------------------------------------
apiVersion: v1 # Версия API
kind: Pod # Объект, который создаем
metadata:
  name: web
  labels: 
    app: web
spec:
  volumes:
  - name: app
    emptyDir: {}
  containers: 
    - name: web
      image: thatsme/web:1.2
      readinessProbe: 
        httpGet: 
          path: /index.html 
          port: 80
---------------------------------------------

Запуск web-pod.yaml
kk apply -f web-pod.yaml 

С ipvs настройки дал, но ipvs не отобразил ничего, ругался на 
root@minikube:~# ipvsadm
Can't initialize ipvs: Permission denied (you must be root)
Are you sure that IP Virtual Server is built in the kernel or as module?

MetalLB
kubectl edit configmap -n kube-system kube-proxy
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true



kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml

vi metallb-config.yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
      - name: default
        protocol: layer2
        addresses:
          - "172.17.255.1-172.17.255.255"


# ---
# apiVersion: metallb.io/v1beta1
# kind: IPAddressPool
# metadata:
#   name: default
#   namespace: metallb-system
# spec:
#   addresses:
#   - 172.17.255.0/24

# ---
# apiVersion: metallb.io/v1beta1
# kind: L2Advertisement
# metadata:
#   name: metallb
#   namespace: metallb-system
# spec:
#   ipAddressPools:
#   - default


kubectl apply -f metallb-config.yaml


sudo ip route add 172.17.255.0/24 via 192.168.49.2

Задание со звездочкой
apiVersion: v1
kind: Service
metadata:
  name: coredns-svc-lb-tcp
  namespace: kube-system
  annotations: 
    metallb.universe.tf/allow-shared-ip: "key-to-share-172.17.255.2"
spec:
  selector:
    k8s-app: kube-dns
  type: LoadBalancer
  loadBalancerIP: 172.17.255.2
  ports:
    - protocol: TCP
      port: 53
      targetPort: 53
      name: dns-tcp
---
apiVersion: v1
kind: Service
metadata:
  name: coredns-svc-lb-udp
  namespace: kube-system
  annotations: 
    metallb.universe.tf/allow-shared-ip: "key-to-share-172.17.255.2"
spec:
  selector:
    k8s-app: kube-dns
  type: LoadBalancer
  loadBalancerIP: 172.17.255.2
  ports:
    - protocol: UDP
      port: 53
      targetPort: 53
      name: dns 

Проверка 
[m.efoshkin@laptop ~]$ nslookup kube-dns.kube-system.svc.cluster.local 172.17.255.2
Server:		172.17.255.2
Address:	172.17.255.2#53

Name:	kube-dns.kube-system.svc.cluster.local
Address: 10.96.0.10


Ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml

kk apply -f nginx-lb.yaml
kk apply -f web-svc-headless.yaml 
kk apply -f web-ingress.yaml

curl 172.17.255.3/web


******
Задание со звездочкой
Установка дащбордов
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

kk get ingress -n kubernetes-dashboard
kk apply -f dashboard/web-ingress.yaml 
kk apply -f dashboard/web-svc-headless.yml