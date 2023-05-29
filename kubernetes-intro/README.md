
Тут находится описание проделанной работы. 


git clone --config core.sshCommand="ssh -i /home/m.efoshkin/.ssh/otus_rsa" git@github.com:otus-kuber-2023-04/efoshkinm_platform.git


Запуск миникуба
minikube start --driver=podman --container-runtime=containerd

Kubernetes Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

Проверить что кластер находится в рабочем состоянии.
kubectl get componentstatuses

Docker file

FROM nginx:stable
RUN   mkdir /app \
      &&  usermod -u 1001 nginx   \
      && rm /etc/nginx/conf.d/default.conf
COPY index.html homework.html /app
COPY homework.conf /etc/nginx/conf.d
WORKDIR /app
EXPOSE 80
CMD ["-g", "daemon off;"]
ENTRYPOINT ["nginx"]

docker build . -t docker.io/efoshkinm/otus/first-docker
docker run -d -p 127.0.0.1:8000:80 --name my_docker docker.io/efoshkinm/otus/first-docker


server {
    listen       80;
    server_name  localhost;
    location / {
        root   /app;
        index  index.html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}

index.html


podman tag docker.io/efoshkinm/otus/first-docker docker.io/efoshkinm/first-docker
podman push docker.io/efoshkinm/first-docker



Создаем под
vi web-pod.yaml
apiVersion: v1 # Версия API
kind: Pod # Объект, который создаем
metadata:
 name: web
 labels: 
   app:web
spec: 
 containers: 
 - name: web
 image: docker.io/efoshkinm/first-docker

 kubectl apply -f web-pod.yaml

Added volumes
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
   image: docker.io/efoshkinm/first-docker
   volumeMounts:
   - name: app
     mountPath: /app

 - name: init
   image: busybox:1.36.1
   command: ['sh', '-c', 'wget -O- https://tinyurl.com/otus-k8s-intro | sh']
   volumeMounts:
   - name: app
     mountPath: /app


kk port-forward --address 0.0.0.0 pod/web 8000:80


Hipster Shop

git clone git@github.com:GoogleCloudPlatform/microservices-demo.git

cd microservices-demo
podman push docker.io/efoshkinm/frontend-hipster:latest 

kk run frontend --image docker.io/efoshkinm/frontend-hipster:latest --restart=Never

Сгенерировать ямл код
 kk run frontend --image docker.io/efoshkinm/frontend-hipster:latest --restart=Never --dry-run -o yaml > frontend-pod.yaml

kk logs frontend
{"message":"Tracing disabled.","severity":"info","timestamp":"2023-05-29T05:15:27.321613115Z"}
{"message":"Profiling disabled.","severity":"info","timestamp":"2023-05-29T05:15:27.321669643Z"}
panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set

goroutine 1 [running]:
main.mustMapEnv(0xc000000000, {0xc0764f, 0x1c})
        /src/main.go:208 +0xb9
main.main()
        /src/main.go:124 +0x5be