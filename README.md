# AWX
Instalacion de awx con awx-operator sobre Kubernetes K3S

Deshabilitar el firewall:

systemctl disable --now firewalld.service

Deshabilitar swap del servidor:

swapoff -a

entrar al archivo /etc/fstab y comentar la siguiente linea:

![image](https://github.com/user-attachments/assets/a693e126-bc52-4cf6-9ff2-6e5fcc273f12)


1- Instalación de kubernetes con K3s. 

curl -sfL https://get.k3s.io | sh -s - --data-dir /opt/k3s --disable traefik

mkdir -p $HOME/.kube

cp -i /etc/rancher/k3s/k3s.yaml $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config

2- Clonar el repositorio de awx-operator y asignar el tag de la version que se instalará de AWX-OPERATOR

git clone https://github.com/ansible/awx-operator.git

cd awx-operator

git tag

git checkout tags/2.19.1

3- Crear el namespace para awx:

kubectl create namespace awx

4- Crear la carpeta playbooks dentro del opt

mkdir /opt/playbooks

4- Aplicar el YAML que contiene la configuracion del pv y el pvc para que la ruta de los proyectos dentro del contenedor /var/lib/awx/projects se comparta con el servidor anfitrion.

kubectl apply -f volumenes.yaml

5- Construir los archivos kustomization.yaml y awx-demo.yaml para realizar la instalacion, primero se debe aplicar el kustomization.yaml y cuando este levante los contenedores editarlo para agregar la linea "- awx-demo.yml" 

kubectl apply -k .

![image](https://github.com/user-attachments/assets/2be94a2b-ec73-4c92-ad14-e1fd145355f9)

6- Para loguearse en la interfaz grafica utilizar el usuario admin y la contraseña se obtiene con el siguiente comando:

kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo

8- Instalacion del ingress para cargar por HTTPS con certificado y DNS:

8.1 crear secreto con el wildcard.crt y wildcard.key

kubectl create secret tls tls-secret --cert=wildcard.crt --key=wildcard.key -n awx

8.2 Aplicar todos los manifiestos presentes en la carpeta balanceador

kubectl apply -f .

8.3 

Documentacion oficial: https://ansible.readthedocs.io/projects/awx-operator/en/latest/installation/basic-install.html
