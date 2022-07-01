# kubeadm-workshop

### Конфигурация стенда  
Для сегодняшней практики необходимо создать 3 вм  
Все в одной приватной сети, на каждую подключен флоатинг адрес  
Конфиг ВМ Standard Line 2CPU / 4Gb Ram / 15Gb универсального диска  
OS Ubuntu 22.04 LTS 64-bit  
**В системе должен быть отключен swap и selinux**  
**Будет не лишним добавить все хосты кластера в /etc/hosts каждого севрера**  
Именование хостов k8s-controller-1, k8s-worker-1, k8s-worker-2  


### Добавляем необходимые репозиториии
```
apt-get update -y  
apt-get install apt-transport-https ca-certificates curl ntp -y  
```

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add  
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"  
```

```
echo "deb [signed-by=/usr/share/keyrings/libcontainers-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_22.04/ /" >> /etc/apt/sources.list  
echo "deb [signed-by=/usr/share/keyrings/libcontainers-crio-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.24:/1.24.1/xUbuntu_22.04/ /" >>  /etc/apt/sources.list 
```

```
mkdir -p /usr/share/keyrings  
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_22.04/Release.key | gpg --dearmor -o /usr/share/keyrings/libcontainers-archive-keyring.gpg  
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.24:/1.24.1/xUbuntu_22.04/Release.key | gpg --dearmor -o /usr/share/keyrings/libcontainers-crio-archive-keyring.gpg  
```

### Устанавливаем runtime
```
apt-get update  
apt-get install cri-o cri-o-runc -y  
```

### Заносим в систему crictl
```
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.24.1/crictl-v1.24.1-linux-amd64.tar.gz --output crictl-v1.24.1-linux-amd64.tar.gz  
tar zxvf crictl-v1.24.1-linux-amd64.tar.gz -C /usr/local/bin  
rm -f crictl-v1.24.1-linux-amd64.tar.gz  
```

### Запускаем cri-o
```
systemctl enable crio
systemctl start crio
systemctl status crio
```

### Проверяем, отключен ли swap
```
swapon -s
cat  /proc/swaps
```

### Устанавливаем компонеты K8S и kubeadm
```
apt-get install kubeadm kubelet kubectl kubernetes-cni -y
```

### Добавляем загрузку модулей overlay и br_netfilter
```
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

#### Загружаем их руками
```
modprobe overlay  
modprobe br_netfilter  
```

### Добавляем настройки sysctl для включения роутинга и фильтрации на бридже  
```
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

#### Применяем настройки sysctl
```
sysctl --system
```

### Инициализируем kubeadm, только на controller (сеть для pod будет 192.168.128.0/17)
```
kubeadm init --pod-network-cidr=192.168.128.0/17 --apiserver-cert-extra-sans=ПЛАВАЮЩИЙ_АДРЕС_k8s-controller-1
```

### Следуя подсказке kubeadm копируем kubeconfig в домашюю директорию пользователя  
```
mkdir -p $HOME/.kube  
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
chown $(id -u):$(id -g) $HOME/.kube/config  
```

**Очень важно сохранить из подсказки kubeadm строку с join воркеров (тут пример, у каждого будут уникальные значения)**  
```
kubeadm join 192.168.0.2:6443 --token ni4l5i.tq9qlmhl9i72g504 \  
	--discovery-token-ca-cert-hash sha256:9cae2c72ad311676e68d09f3259115ed04a2801f7e8db23d099a8cd338dbb2f9  
```
 
#### Проверим состояние кластера
```
kubectl get nodes  
kubectl get po -A  
```

**Повторяем на воркерах процесс установки, пропуская фазу kubeadm init**

### Добавляем воркеры в кластер
**Выполянем kubeadm join, строку, которую сгенерил kubeadm**  

#### На текущем этапе у нас в кластере должно быть активных 3 хоста.  

### Устанавливаем CNI Calico
```
kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml  
kubectl apply -f https://raw.githubusercontent.com/neotkm/kubeadm-workshop/main/custom-resources-calico.yaml  
```

### Изменяем ConfigMap CoreDNS
```
kubectl apply -f https://raw.githubusercontent.com/neotkm/kubeadm-workshop/main/coredns-config-map.yaml
```

#### Находим поды CoreDNS и удаляем их, чтобы они получили новый ConfigMap
```
kubectl get po -n kube-system | grep coredns  
kubectl delete po -n kube-system coredns-ИМЯ_ПОДА
```

### Развернем K8S dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml  
kubectl apply -f https://raw.githubusercontent.com/neotkm/kubeadm-workshop/main/dashboard-adminuser.yaml  
kubectl -n kubernetes-dashboard create token admin-user  
```
**Выхлоп после create token сохраняем себе**  

#### Получаем доступ к kubernetes-dashboard  
Создаем директорию для конфига 
```
mkdir -p $HOME/.kube
```
Копируем с k8s-controller-1 конфиг
```
scp root@Плавающий_IP_ADDRESS_Контроллера:~/.kube/config ~/.kube/config
```
Необходимо в файле `~/.kube/config` изменить строку server: https://*.*.*.*:6443, заменив указанный там ip address на плавающий адрес k8s-controller-1  

После этого, запускаем `kubectl proxy` , это позволит нам обращаться к приложениям запущенным в кластере, но не опубликованным наружу  

Открываем ссылку
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```
Отмечаем чекбокс "Token" и вставляем в форму токен, который мы сохраняли после `create token admin-user` , мы должны попасть в UI кластера

### Установим в кластер metrics-server
```
kubectl apply -f https://raw.githubusercontent.com/neotkm/kubeadm-workshop/main/metrics-server.yaml

```
Через несколько минту у pod-ов и хостов кластера, появятся метрики по утилизации ресурсов.

### Развернем ingress-nginx controller (c net=host)
```
kubectl apply -f https://raw.githubusercontent.com/neotkm/kubeadm-workshop/main/deploy-ingress-nginx.yaml
```

### Развернем демо приложение, для демонстрации работы ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/neotkm/kubeadm-workshop/main/demo-ingress.yaml
```
#### Проверим работу приложения и ingress
```
while true; do sleep 0.5; curl http://Плавающтий_IP_ADDRESS_Воркера/ -H 'Host: example.com'; echo -e '\n'$(date);done
```
