# Kind And Ansible Deployment

## Requisitos

- Kind
- Cloud Provider KIND
- Kubectl
- Helm
- Ansible

### Kind

> Requisitos previos: Docker

Instale Kind de forma local utilizando el script en carpeta `kind` use GIT BASH si utiliza Windows.

El script instala `Local Registry` y `Kind` ya configurado, debe editar el script para asignar un nombre a su Cluster, por defecto tiene el nombre `kind-cluster`.

```bash
sh ./kind/kind-with-registry.sh
```

Fuente: [Kind](https://kind.sigs.k8s.io/docs/user/local-registry/)

### Cloud Provider KIND

> Requisitos previos: [GO](https://go.dev/doc/install)

Necesario si va a utilizar `LoadBalancer` el script de creación de cluster tiene incorporado `extraPortMappings`, puede usar ambas opciones en su entorno local.

```bash
go install sigs.k8s.io/cloud-provider-kind@latest
```

### Kubectl

```sh
# Ubuntu
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl

# Mac OS
brew install

# Windows
curl.exe -LO "https://dl.k8s.io/release/v1.32.0/bin/windows/amd64/kubectl.exe"
```

### Helm

```sh
# Ubuntu
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# Mac OS
brew install helm

# Windows
# Utilice WSL2 he instale pra linux (Ubuntu) especificada mas arriba.
```

### Ansible

```sh
# Ubuntu
sudo apt install ansible

# Mac OS
brew install ansible

# Windows
# Siga los pasos de Ubuntu sobre el subsistema de linux
```

Instalar dependencias

> Instalar Python y Pip en su equipo, si utiliza windows instalar ambas en el sub-sistema de linux

```sh
# Ubuntu
pip install kubernetes-validate pyyaml kubernetes

# Mac OS
pip3 install kubernetes-validate pyyaml kubernetes

# Windows
# Siga los pasos de Ubuntu sobre el subsistema de linux
```

## Crear archivos de despliegue

Use los archivos de ejemplo en la carpeta Local

### Preparar carga

Como esto es un entorno local es necesario cargar a nuestro registro local las imagenes y archivos helm para el despliegue.

En el caso de los depliegue de chart de helm debe ingresar al proyecto posicionar sobre la carpeta chart y ejecutar los siguientes comandos:

#### Registro Local (Kind-Registry)

Comprime los template y sus valores por defecto ofreciendonos un archivo para cargar, la versión asignada la obtiene desde el archivo `Chart.yml`

```bash
helm package .
```

Suba a su contenedor de registro local la configuracion de Helm para el proyecto

```bash
helm push project-service-0.0.1.tgz oci://localhost:5001/helm
```

Siga los pasos de cada proyecto para crear una imagen local y posteriormente ejecuta los siguientes comandos para cargarlos a contenedor de registros local

```bash
docker tag proyect-image localhost:5001/proyect-image:0.0.1

docker push localhost:5001/proyect-image:0.0.1
```

> La version del tag depende del manejo de versiones que utilizará, puede utilizar la version de semantic release o calc release

### Despligue inicial

#### Windows (Consideraciones)

siga estos pasos desde la consola de su WSL2, posicionado sobre este proyecto, agregue a sus variables de entorno la referencia de la instalacion de kubectl de windows.

Instalar ingress controller

Esta es una versión obtenida desde [Kind](https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml), ha sido modificada para incorporar `extraPortMappings` por defecto y tambien para administrar la version de nginx que se instale.

```bash
ansible-playbook ansible/nginx.yml -vvv
```

### Despliegue de servicios

Servicios:

- Instalar Namespaces

  ```bash
  ansible-playbook ansible/namespaces.yml -vvv
  ```

- Instalar secrets de servicios

  ```bash
  ansible-playbook ansible/secrets.yml -vvv
  ```

- Instalar servicios

  ```bash
  ansible-playbook ansible/services.yml -vvv
  ```

## Consideraciones Generales

Como es un ambiente local los template de ingress no deben contener referencias a TLS o a los Hosts
