[![1](https://github.com/rozoandrescamilo/Smart-Contract-para-consultar-estados-de-una-Purchase-Order/blob/main/img/1.jpg?raw=true "1")](https://github.com/Smart-Contract-para-consultar-estados-de-una-Purchase-Order/blob/main/img/1.jpg?raw=true "1")

# Investigación para servicios de Lacchain usando Kubernetes

Este documento se basará en la documentación proporcionada en la guía oficial de Lacchain, disponible en [https://lacnet.lacchain.net/documentation/](https://lacnet.lacchain.net/documentation/ "https://lacnet.lacchain.net/documentation/").

 - [Instalar un nodo de la red de Lacchain](#instalar-un-nodo-de-la-red-de-lacchain)
  - [Requisitos previos](#requisitos-previos)
  - [Pasos de instalación](#pasos-de-instalación)
    - [Paso 1: Configuración del entorno](#paso-1-configuración-del-entorno)
    - [Paso 2: Configuración de variables de entorno](#paso-2-configuración-de-variables-de-entorno)
    - [Paso 3: Desplegar nuevo nodo](#paso-3-desplegar-nuevo-nodo)
    - [Paso 4: Configuración del nuevo nodo](#paso-4-configuración-del-nuevo-nodo)

 - [Instalar nodo IPFS (InterPlanetary File System)](#instalar-nodo-ipfs-interplanetary-file-system)
  - [Requisitos mínimos del sistema](#requisitos-mínimos-del-sistema)
  - [Instalación](#instalación)
    - [Paso 1: Configuración del repositorio](#paso-1-configuración-del-repositorio)
    - [Paso 2: Instalación de Nodo](#paso-2-instalación-de-nodo)
    - [Paso 3: Verificar conexión](#paso-3-verificar-conexión)
  - [Enviar documento a IPFS](#enviar-documento-a-ipfs)

## Instalar un nodo de la red de Lacchain

La siguiente investigación tiene como objetivo proporcionar una guía paso a paso para instalar un nodo de la red de Lacchain utilizando Kubernetes.

Se proporcionarán instrucciones para implementar nodos en Kubernetes utilizando HELM. Este proceso implicará la ejecución desde una máquina local hacia un servidor remoto, estableciendo la comunicación entre ambas a través de HELM. La instalación utilizando manifiestos de Kubernetes es compatible con Google Kubernetes Engine.

### Requisitos previos

#### Requisitos mínimos del sistema

Antes de comenzar la instalación, asegúrese de tener los siguientes requisitos previos:

[![1](https://github.com/rozoandrescamilo/Investigacion-Lacchain-usando-Kubernetes/blob/main/img/1.png?raw=true "1")](https://github.com/Investigacion-Lacchain-usando-Kubernetes/blob/main/img/1.png?raw=true "1")

Es necesario habilitar los siguientes puertos de red en la máquina en la que se desplegará el nodo:

Nodo Besu :
- Puerto 60606 : TCP/UDP – Para establecer comunicación p2p entre nodos.
- Puerto 4545 : TCP – Para establecer comunicación RPC. Este puerto se utiliza para aplicaciones que se comunican con el nodo y pueden filtrarse a Internet.

Nginx :
- Puerto 80 : TCP: para establecer comunicación RPC con el modelo de gas.


Para instalar un nodo en un clúster de Kubernetes, es necesario instalar dos herramientas:

Kubectl y Helm. Se instalan en la máquina local desde la cual se realizará la instalación del nodo en el clúster de Kubernetes. Asegúrese de seguir las instrucciones proporcionadas para instalar correctamente tanto Kubectl como Helm en su máquina local antes de continuar con la instalación del nodo.

Documentación Instalación Kubectl:  [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/ "https://kubernetes.io/docs/tasks/tools/")

Documentación Instalación Helm: [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/ "https://helm.sh/docs/intro/install/")


### Pasos de instalación

Siga los siguientes pasos para instalar un nodo de la red de Lacchain utilizando Kubernetes:

#### Paso 1: Configuración del entorno
Asegúrese de tener acceso a su terminal de línea de comandos y su clúster de Kubernetes.

Clone el repositorio oficial de Lacchain en su máquina local utilizando el siguiente comando:

`git clone https://github.com/LACNetNetworks/besu-networks`

Cambie al directorio del repositorio clonado:

`cd besu-networks/helm/`

#### Paso 2: Configuración de variables de entorno

Hay tres tipos de valores que corresponden a los cuatro tipos de nodos: bootnode.yml , validator.yml ,  writer.yml. Los valores que tienes que configurar están en la sección de despliegue. Estos son los siguientes:

- **red :** Tipo Red –  red principal | red-de-protesta-abierta | red de protesta .
- **typenode :** Tipo de Nodo – escritor | validador | nodo de arranque
- **publicIP:** Entrada de IP pública de TCP.
- **p2p – host :** salida de IP pública P2P.
- **p2p – puerto :** PUERTO P2P – Predeterminado (60606).
- **workerName :** Nombre del nodo trabajador donde siempre se instalará el pod.
- **dnsName :** nombre de dominio de la organización (por ejemplo, lacchain.com).
- **nodeName :** nombre que desea para su nodo en la herramienta de monitoreo de red.
- **nodeEmail :** dirección de correo electrónico que desea registrar para su nodo en la herramienta de monitoreo de red. Es una buena idea proporcionar el correo electrónico del contacto técnico identificado o identificado en el formulario de registro como parte del proceso de incorporación.

Se deben establecer los valores de entorno en helm en el archivo /helm/values/writer.yml. 

[![2](https://github.com/rozoandrescamilo/Investigacion-Lacchain-usando-Kubernetes/blob/main/img/2.png?raw=true "2")](https://github.com/Investigacion-Lacchain-usando-Kubernetes/blob/main/img/2.png?raw=true "2")

**Entrada de IP pública TCP:** Genera una IP pública estática en tu proveedor de cloud. Luego reemplace la ip pública en el manifiesto del servicio de balance de carga (loadBalancerIP). finalmente actualice la variable de entorno publicIP con esta IP.

**IP pública de salida P2P:** Tráfico p2p saliente para sincronizar los nodos besu. Esta es la IP autorizada para la red. Por lo tanto, el pod debe estar siempre instalado en el mismo nodo trabajador para que la IP no cambie. Obtenemos el nombre y la IP de los nodos del cluster con el siguiente comando:

`kubectl get nodes -o wide`

Elegimos un worker y actualizamos el valor "nodeName" en el manifiesto del pod que vamos a desplegar. finalmente actualizamos la variable de entorno p2p - host con la IP externa del worker .

Nota: Validamos que el pod se ha desplegado en el worker seleccionado con el siguiente comando:

` kubectl get pod -o wide`

#### Paso 3: Desplegar nuevo nodo

Para desplegar el nodo escritor se deberá ejecutar el siguiente comando (este despliegue sólo es compatible con Google Kubernestes Engine GKE):

`helm install <chart-name>  ./charts/besu-node --namespace  <namespace-name> --create-namespace --values ./values/writer.yml`

Desplegar nuevo nodo en la red Mainnet-Omega:

`helm install lacnet-writer-1 ./charts/besu-node --namespace  lacchain-main-net --create-namespace --values ./values/writer.yml`

Al final de la instalación, si todo ha funcionado, se creará un servicio BESU gestionado por Systemctl con el estado Running. Los objetos adicionales creados son namespace, service load balancer, configmap y volume.

No olvidar anotar el "enode" del nodo:

`curl -X POST --data '{"jsonrpc":"2.0","method":"net_enode","params":[],"id":1}' http://<PUBLIC_IP>:4545`

Output:

```json
"enode://d837cb6dd3880dec8360edfecf49ea7008e50cf3d889d4f75c0eb7d1033ae1b2fb783ad2021458a369db5d68cf8f25f3fb0080e11db238f4964e273bbc77d1ee@104.197.188.33:60606"
```

Recuerde que debe proporcionar el nodo para que se le conceda el permiso, tal y como se indica en el proceso de concesión de permisos. También como parte del proceso de autorización, necesitamos que nos proporciones la dirección de tu nodo. Por favor, obténgala ejecutando lo siguiente:

`curl -X POST --data '{"jsonrpc":"2.0","method":"eth_coinbase","params":[],"id":53}' http://<PUBLIC_IP>:4545`

Output

```json
{
  "jsonrpc" : "2.0",
  "id" : 53,
  "result" : "0xa08d3d8f68ba47deb401769e5ed39ff283e60a80"
}
```

#### Paso 4: Configuración del nuevo nodo

- Configuración del archivo de nodo de Besu

La configuración por defecto debería funcionar para todos. Sin embargo, dependiendo de sus necesidades y conocimientos técnicos puede modificar la configuración de su nodo en valores carpeta writer.yml bootnode.yml validator.yml, para el acceso RPC o autenticación. Consulte la documentación de referencia.

- Comprobación de la conexión

Una vez obtenido el permiso, puede comprobar si su nodo está conectado correctamente a la red. Comprueba que el nodo ha establecido las conexiones con los peers:

`curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' http://<PUBLIC_IP>:4545`

Output:

[![3](https://github.com/rozoandrescamilo/Despliege-de-nodo-local-con-Lacchain/raw/main/img/12.png?raw=true "3")](https://github.com/rozoandrescamilo/Despliege-de-nodo-local-con-Lacchain/raw/main/img/12.png?raw=true "3")


Ahora puede comprobar si el nodo está sincronizando bloques obteniendo el log:

`kubectl logs <pod name> -c <container name> -f -n <namespace>`

Ejemplo de Nodo escritor en la red  Mainnet-Omega

`kubectl logs besu-node-lacnet-writer-1-0 -c lacnet-writer-1-besu  -f -n lacchain-main-net`

Debería obtener algo como esto:

[![4](https://github.com/rozoandrescamilo/Despliege-de-nodo-local-con-Lacchain/raw/main/img/13.png?raw=true "4")](https://github.com/rozoandrescamilo/Despliege-de-nodo-local-con-Lacchain/raw/main/img/13.png?raw=true "4")

- Actualización del nodo

Si necesita actualizar el nodo, intente volver a desplegar el nodo besu: 

`helm upgrade <chart-name> ./charts/besu-node --namespace  <namespace-name>  --values ./values/writer.yml`

Si alguna de estas dos comprobaciones no funciona, intente reiniciar el servicio besu: 

`kubectl delete pod <pod name> -n <namespace>`
`kubectl delete pod besu-node-lacnet-writer-1-0  -n lacchain-main-net`



## Instalar nodo IPFS (InterPlanetary File System)

A continuación encontrará instrucciones para el despliegue de nodos ipfs en Kubernetes utilizando Helm. Esto implica que se ejecutará desde una máquina local en un servidor remoto. La máquina local y el servidor remoto se comunicarán a través de Helm. La instalación con manifiestos kubernetes es compatible con Google Kubernetes Engine .

### Requisitos mínimos del sistema

Características de hardware recomendadas para el nodo IPFS:

[![5](https://github.com/rozoandrescamilo/Investigacion-Lacchain-usando-Kubernetes/blob/main/img/5.png?raw=true "5")](https://github.com/Investigacion-Lacchain-usando-Kubernetes/blob/main/img/5.png?raw=true "5")

Es necesario habilitar los siguientes puertos de red en la máquina en la que se desplegará el nodo:

Peers IPFS:

- 4001: TCP - Puerto para establecer Comunicación p2p con otros peers.

- 5001: TCP - Servidor API.

- 8080: TCP - Servidor Gateway.

Para instalar un nodo en un clúster de Kubernetes, es necesario instalar dos herramientas:

Kubectl y Helm. Se instalan en la máquina local desde la cual se realizará la instalación del nodo en el clúster de Kubernetes. Asegúrese de seguir las instrucciones proporcionadas para instalar correctamente tanto Kubectl como Helm en su máquina local antes de continuar con la instalación del nodo.

### Instalación

Siga los siguientes pasos para instalar un nodo IPFS de la red Lacchain utilizando Kubernetes:

#### Paso 1: Configuración del repositorio
Asegúrese de tener acceso a su terminal de línea de comandos y su clúster de Kubernetes.

Clone el repositorio oficial de IPFS de Lacchain en su máquina local utilizando el siguiente comando:

`git clone https://github.com/LACNetNetworks/ipfs.git`
`cd ipfs/helm/`

#### Paso 2: Instalación de Nodo

**Variable de valores**

Hay cuatro tipos de valores ipfs.yml.

Los valores que debe establecer se encuentran en la sección deploy. Son los siguientes:

Valores:

- **logging:** LOGGING - Nivel de logging IPFS (INFO, DEBUG) - por defecto INFO.
- **publicIP:** Entrada de IP pública TCP.

**Establecer valor en la variable de entorno**

- **Entrada IP Pública TCP:** Generar una IP pública estática en su proveedor de nube. A continuación, sustituya la IP pública en el manifiesto del servicio de equilibrio de carga (loadBalancerIP). por último, actualice la variable de entorno publicIP con esta IP.
- **red:** Escriba Network - main-net | open-protest-net .

**Desplegar el nuevo nodo**

Es necesario ejecutar el siguiente comando para desplegar un Nodo IPFS:

Nota: Este despliegue sólo es compatible con Google Kubernestes Engine GKE.

`helm install <nombre-del-nodo> ./charts/ipfs-node --namespace <nombre-del-espacio-de-nombres> --create-namespace --values ./values/ipfs.yml`

Ejemplo de despliegue del nodo IPFS en la red Mainnet-Omega:

`helm install bid ./charts/ipfs-node --namespace lacnet-ipfs --create-namespace --values ./values/ipfs.yml`

#### Paso 3: Verificar conexión

Puede comprobar si su nodo ipfs está conectado a la red LAC Net. Compruebe que el nodo ha establecido las conexiones con los pares:

Para Omega MainNet usar el comando:

`curl http://<PUBLIC_IP>:8080/ipfs/QmT7doZQU171dk3XmixagjVrT73dj8MP4XXqLj6EBTWyHA`

Debería obtener un resultado como este:

```json
bienvenido a ipfs LACNet
```

Para Open ProTestNet usar el comando:

` curl http://<PUBLIC_IP>:8080/ipfs/QmeLSZuyZK8Gzg5Am2miiPP4vt4cX84aWUeu6uzt96VbXT`

Debería obtener un resultado como este:

```json
bienvenido a ipfs LACNet (open-protest)
```

Ahora puede comprobar si el nodo se está ejecutando obteniendo el log:

`kubectl logs <nombre del nodo> -f --namespace <nombre del espacio de nombres>`

[![6](https://github.com/rozoandrescamilo/Despliege-de-nodo-local-con-Lacchain/raw/main/img/18.png?raw=true "6")](https://github.com/rozoandrescamilo/Despliege-de-nodo-local-con-Lacchain/raw/main/img/18.png?raw=true "6")

Si necesita actualizar el nodo, intente volver a desplegar el nodo ipfs: 

`helm upgrade <nombre-carta> ./charts/ipfs-node --namespace <nombre-espacio-de-nombres> --values ./values/ipfs.yml`

Si alguna de estas dos comprobaciones no funciona, intenta reiniciar el servicio ipfs:

`kubectl delete pod <nombre del pod> -n <espacio de nombres>`

### Enviar documento a IPFS

**Requisitos**

- Nodo IPFS
- Nodejs > 10.x
- [ipfs-http-cliente](https://www.npmjs.com/package/ipfs-http-client "ipfs-http-cliente")
- Repositorio instalado https://github.com/LACNetNetworks/ipfs.git

Muévase a la  carpeta de ejemplo:

`cd sample`

Luego instala las dependencias.

`npm install`

El siguiente código permite registrar una [credencial de academia](https://github.com/LACNetNetworks/ipfs/blob/master/sample/academic.json "credencial de academia") en ipfs y luego registrar el hash de ipfs en el blockhain.

```javascript
const express = require('express');
const ipfsClient = require('ipfs-http-client');

const ipfs = ipfsClient('http://localhost:5001');
const app = express();

app.use(express.json());

app.post('/addCredential', async (req, res) => {
    const data = req.body;

    const fileHash = await addFile(data);
    return res.send(`http://localhost:8080/ipfs/${ fileHash }`);
});

const addFile = async (file) => {
    const filesAdded = await ipfs.add(JSON.stringify(file));
    return filesAdded[0].hash;
}
app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

Arrancamos el servidor en el puerto 3000.

`node index.js`

Registramos nuestra credencial en IPFS

`curl --location --request POST 'localhost:3000/addCredential' \`

```json
--header 'Content-Type: application/json' \
--data-raw '{
	"@context": [
		"https://www.w3.org/2018/credentials/v1",
		"https://www.lacchain.net/credentials/library/education/4e6c312cd8e6b18116fe3fd2e9b6e5df810afe0a716c1c511ef6c19cb8554578/v1"
	],
	"id": "d49ec380-49eb-474f-8128-a572e8b29278",
	"type": [
		"VerifiableCredential",
		"Certificate"
	],
	"issuer": "did:ethr:lacchain:0xadf1702b76419f428014d1386af487b2d8145f83",
	"issuanceDate": "2020-10-21T15:49:00.057Z",
	"expirationDate": "2025-10-21T15:49:00.057Z",
	"credentialSubject": {
		"id": "did:ethr:lacchain:0x48007072061dc756e5a2ecf15cf2c2bcc091de52",
		"givenName": "George",
		"familyName": "Walch",
		"email": "gw@town.org",
		"holds": {
			"category": "Diploma",
			"industry": "Computer Science",
			"skillset": "Blockchain",
			"course": "Introducción a LACChain Besu",
			"description": "Curso introductorio de despliegue de nodos en LACChain Besu para desarrolladores",
			"url": "https://aula.blockchainacademy.cl/p/introduccion-a-lacchain",
			"duration": 40,
			"modality": "virtual",
			"location": null
		}
	},
	"evidence": true,
	"credentialStatus": {
		"id": "0x4185Dab0662ccDa3D3F35779578a4242bb89Db37",
		"type": "SmartContract"
	},
	"proof": [
		{
			"id": "did:ethr:lacchain:0xadf1702b76419f428014d1386af487b2d8145f83",
			"type": "EcdsaSecp256k1Signature2019",
			"proofPurpose": "assertionMethod",
			"verificationMethod": "0x7a746D34754C14EB3eb1F214BD0EA23a1A18Be7A",
			"proofValue": "0x0b8d75810bf30fd221ddd6892e9e50b1c63240aba4767a9af2bfb8d8e5944e7b169f78d99f3174055de5a7fcbb65d1367c900a19302c5409a01d77339001d7181b"
		},
		{
			"id": "did:ethr:lacchain:0xf0c0ee53386b8463ff3e58203d45d982058b7917",
			"type": "EcdsaSecp256k1Signature2019",
			"proofPurpose": "assertionMethod",
			"verificationMethod": "0x7a746D34754C14EB3eb1F214BD0EA23a1A18Be7A",
			"proofValue": "0xb1d3eb341fef023b28a9e62bb6ba66ca5b473872249bd6724afcbc71a0d08a8e5a1fd2a325d5591e1380e94404921b90d6589b9366e9c2c638b2683e7c230fcd1b"
		},
		{
			"id": "did:ethr:lacchain:0xaa799564d54356cc754bd5d126101602f1d01ced",
			"type": "EcdsaSecp256k1Signature2019",
			"proofPurpose": "assertionMethod",
			"verificationMethod": "0x7a746D34754C14EB3eb1F214BD0EA23a1A18Be7A",
			"proofValue": "0x4361ba3b2e1a392858e5a6da1a4a948f1f73107849951e4167d9ad89cc0a35bf593d774e9e3ea9cf3ce2b7133c9c8ae1c6d3c2730ff0ec5c20a602cc1b61f21c1b"
		}
	]
}'
```

Como resultado el hash ipfs  QmVPwyet87XRotrdyj1jrhaoXfiav9dupZsTkWmPuBjhZj  que puede consultar en la puerta de enlace IPFS [http://localhost:8080/ipfs/QmVPwyet87XRotrdyj1jrhaoXfiav9dupZsTkWmPuBjhZj](http://localhost:8080/ipfs/QmVPwyet87XRotrdyj1jrhaoXfiav9dupZsTkWmPuBjhZj "http://localhost:8080/ipfs/QmVPwyet87XRotrdyj1jrhaoXfiav9dupZsTkWmPuBjhZj")
