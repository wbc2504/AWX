# AWX
Instalacion de awx con awx-operator sobre Kubernetes K3S

1- Instalación de kubernetes con K3s. 

curl -sfL https://get.k3s.io | sh -s - --data-dir /opt/k3s --disable traefik
mkdir -p $HOME/.kube
cp -i /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

2- Clonar el repositorio de awx-operator.

git clone https://github.com/ansible/awx-operator.git

3- Crear el namespace para awx:

kubectl create namespace awx

4- Aplicar el YAML que contiene la configuracion del pv y el pvc para que la ruta de los proyectos dentro del contenedor /var/lib/awx/projects se comparta con el servidor anfitrion

5- Construir los archivos kustomization.yaml y awx-demo.yaml para realizar la instalacion, primero se debe aplicar el kustomization.yaml y cuando este levante los contenedores editarlo para agregar la linea "- awx-demo.yml" 

6- Para loguearse en la interfaz grafica utilizar el usuario admin y la contraseña se obtiene con el siguiente comando:

kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo
