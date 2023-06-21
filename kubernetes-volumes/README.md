

Установка kind
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind create cluster
export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"

Создание манифестов и проверка
vi minio-statefulset.yaml 
kk get no
kk apply -f minio-statefulset.yaml 
kk get po
kk get pv
vi minio-headless-service.yaml
kk apply -f minio-headless-service.yaml 
kk get svc
kk get sts
kk get pvc
kk get pv
kk describe sts minio 
kk describe pv pvc-0cb125d0-7d94-403c-b93a-831d1bd9edc5 
kk describe pvc data-minio-0 
kk describe po minio-0 
vi minio-secrets.yaml

Создаем секреты
kk apply -f minio-secrets.yaml 
kk get secrets minion-secrets 
kk describe secrets minion-secrets 
kk get secret
kk describe secrets minion-secrets 
kk get sts
kk describe sts minio 
kk apply -f minio-statefulset.yaml 
kk describe sts minio 
kk get po
kk exec -it minio-0 -- bash
kk exec -it minio-0 -- sh


