# <img width="70" height="40" alt="image" src="https://github.com/user-attachments/assets/5a96d25e-157e-439c-8887-7696d8fbd23a" /> Instalacion de AWX

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

5- Aplicar el YAML que contiene la configuracion del pv y el pvc para que la ruta de los proyectos dentro del contenedor /var/lib/awx/projects se comparta con el servidor anfitrion.

kubectl apply -f volumenes.yaml

6- Construir los archivos kustomization.yaml y awx-demo.yaml para realizar la instalacion, primero se debe aplicar el kustomization.yaml y cuando este levante los 2 contenedores del operador editarlo para descomentar la linea "- awx-demo.yml" y volver a aplicar:

kubectl apply -k .

Nota: kustomization.yaml y awx-demo.yaml deben estar en la misma ruta

![image](https://github.com/user-attachments/assets/2be94a2b-ec73-4c92-ad14-e1fd145355f9)

7- Instalacion del ingress para cargar por HTTPS con certificado y DNS:

7.1 crear secreto con el wildcard.crt y wildcard.key

kubectl create secret tls tls-secret --cert=wildcard.crt --key=wildcard.key -n awx

7.2 Aplicar todos los manifiestos presentes en la carpeta balanceador

kubectl apply -f .
kubectl apply -f https://raw.githubusercontent.com/nginx/kubernetes-ingress/v4.0.0/deploy/crds.yaml

7.3 Aplicar el manifiesto nginx-ingress.yml

kubectl apply -f nginx-ingress.yml

Nota: es importante que el ingress pertenezca al mismo namespace que el servicio relacionado en  este mismo ingress.

8- Agregar el DNS awx-ansible.servientrega.com apuntando a la ip del servidor donde se despliego AWX o quemar en el archivo host de la maquina windows que posee el navegador:

192.168.100.3 awx-ansible.servientrega.com

9- Loquearse con el usuario admin y la contraseña se obtiene ejecutando el comando:

kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo

Documentacion oficial: 
- https://ansible.readthedocs.io/projects/awx-operator/en/latest/installation/basic-install.html
- https://docs.nginx.com/nginx-ingress-controller/installation/installing-nic/installation-with-manifests/


