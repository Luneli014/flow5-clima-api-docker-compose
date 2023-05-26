# flow6-clima-api-docker-compose
Este repositorio muestra una estación climática que obtiene los datos climáticos por API ademas de tener un escuchador MQTT local. Se hace uso de un broker publico para darle un carácter colectivo. Este ejercicio funciona mejor con varios participantes reportando a la vez, pues se grafícan los datos de todos.

## Introducción

### Descripción

El flow 5 es una estación climática que muestra los datos locales de temperatura y humedad via MQTT, ya sea de forma manual con terminal o con un micro controlador. Adicionalmente cuenta con una sección que toma la misma información tomada de [Open Weather Map](https://openweathermap.org/) via API.

### Alcances

Este ejercicio requiere muestra la información de un unico usuario, usa un un broker local, hace uso de la API gratuita de [Open Weather Map](https://openweathermap.org/).

## Requisitos
### Material Necesario

Para realizar este flow necesitas lo siguiente

- [Ubuntu 20.04](https://releases.ubuntu.com/20.04/)
- [Docker Engine](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)
- [NodeRed](https://nodered.org/docs/getting-started/local)
- [Nodos Dashboard](https://flows.nodered.org/node/node-red-dashboard)

### Servicios

Necesitas una cuenta gratuita del siguiente servicio
- [Open Weather Map](https://openweathermap.org/)

### Material de referencia

En los siguientes enlaces puedes encontrar cursos en la plataforma de edu.codigoiot.com que te permitirán realizar las configuraciones necesarias

- [Instalación de Virutal Box y Ubuntu 20.04](https://edu.codigoiot.com/course/view.php?id=812)
- [Introducción a Docker](https://edu.codigoiot.com/course/view.php?id=996)
- [Aplicacion multicontenedor de servidor IoT con Docker compose](https://edu.codigoiot.com/mod/lesson/view.php?id=3889&pageid=3804&startlastseen=no)
- [Servidor del Internet de las Cosas con nodeRed](https://edu.codigoiot.com/course/view.php?id=997)



## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas lo siguiente

1. Docker Engine.
2. NodeRed por Docker Compose
3. Contenedor de NodeRed con el volumen de data activado
4. Contenedor de Mosquitto con el volumen de configuración activado. Configurar el archivo ```mosquitto.conf``` con un listener en el puerto ```1883``` para todas las IPs ```0.0.0.0```.
5. Tu usuario de linux debe ser parte del grupo sudoers y del grupo docker
    - Puedes comprobar que tu usuario está en ambos grupos con el comando 
    
        ```
        groups $USER
        ```

    - En caso de que tu usuari no forme parte de dichos grupos, puedes arreglarlo con los siguientes comandos
        ``` 
        su
        sudo usermod -aG sudo newuser
        exit
        ```
### Instrucciones de preparación de entorno

Para arrancar el entorno necesario, puedes usar los siguientes comandos.

1. Comprueba que los contenedores de Mosquitto y nodeRed estén funcionado. Puedes comprobarlo con el comando ```docker ps -a```. En caso de que tus contenedores no estén funcionando, puedes arrancarlos con el comando ```docker start $(docker ps -a -q)```.
2. Comprueba que tengas acceso a la API de Open Weather con la siguiente consulta desde cualquier navegador ```https://api.openweathermap.org/data/2.5/weather?lat=[latitud]&lon=[longitud]&appid=[api_key]&units=metric```. Agrega tu API Key, latitud y longitud de tu ubicación geográfica sin corchetes. Deberías ver un JSON con los datos del clima.
3. Asegurate de tener instalados los nodos ```node-red-dashboard``` en nodeRed.
4. Importa el archivo flows.json a nodeRed y haz clic en el boton **Deploy**.
5. Comprueba que el nodo MQTT apunte al servidor de tu broker local. Si usas Docker Compose, usa el nombre de la aplicación de Mosquitto usado en el archivo compose.yaml como nombre de dominio, en nuestro caso, ```mosquitto```. Si usas una instalación local, usa ```localhost``` o ```127.0.0.1```. Si usas un broker publico usa preferentemente la IP del broker en lugar del nombre de dominio.
6. Asegurate que los nodos function contienen el código correcto.
    Nodo Temperatura

    ``
    msg.payload = msg.payload.temp;
    msg.topic = "Temperatura";
    return msg;
    ``
3. Comprueba que tu broker mosquitto sea accesible desde el exterior del contenedor. La forma fácil de hacerlo, es verificar que el nodo MQTT del flow indique ```conectado```.

### Instrucciónes de operación

1. Dirígete al dashboard en [localhost:1880/ui](http://locahost:1880/ui/)
2. Espera al menos 2 minutos despues de haber importado el flow y haber hecho clic en el botón **Deploy** para observar








Estas son las instrucciones para crear el flow desde cero. Este repositorio incluye el archivo JSON con el Flow funcional.

1. Crear un nuevo flow
2. Agregar un nodo mqtt
	- Agregar un nuevo broker y agregar únicamente la url
	- Server: ```mosquitto``` (o el nombre de tu aplicaciòn)
	- Tema: ```codigoIoT/mqtt/clima```
	- Output: a String
3. Agregar un nodo JSON. Siempre convertir a JavaScript Object
4. Agregar dos nodos function

Nodo function temperatura

```
msg.payload = msg.payload.temp;
msg.topic = "Temperatura";
return msg;
```
Nodo function humedad

```
msg.payload = msg.payload.hum;
msg.topic = "Humedad";
return msg;
```

5. Crear una pestaña y dos grupos de informacion
- Pestaña: Clima local
- Grupo1: Indicadores
- Grupo2: Gráfica
6. Agregar 2 nodos gauge y configurarlos
- Asociarlos al grupo indicadores
- Determine etiquetas y rangos
7. Agregar el nodo chart
- Asociarlos al grupo indicadores
8. Hacer clic en Deploy y consultar el [Dashboard](http://localhost:1880/ui)
9. Enviar un mensaje MQTT al broker local con los valores de temperatura y humedad como temp y hum. Ejemplo:

    ```
    docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":19,"hum":51}'
    ```



### Instrucciones de preparación del entorno

Para ejecutar este flow, es necesario lo siguiente.
1. Detener el contenedor de Mosquitto.

    ```
    docker stop [id_contenedor]
    ```

2. Crear el directorio donde se colocará el archivo msoquitto.conf

    ```
    mkdir -p ~/DockerVolumes/Mosquitto/config
    ```

3. Agregar al directorio creado el archivo mosquitto.conf ubicado en este repositorio.
4. Detener Docker Compose.

    ```
    docker compose stop
    ```
5. Levantar de nuevo Docker Compose

    ```
    docker compose up -d
    ```

6. Arrancar el contenedor de NodeRed con el comando
        
    ```
    docker start $(docker ps -a -q)
    ```

2. Dirigirse a [localhost:1880](http://localhost:1880/)
3. Importar el flow desde el repositorio
4. Hacer clic en el boton Deploy

### Instrucciones de operación

Para ver el resultado de este flow es necesario enviar un mensaje MQTT. Puedes hacerlo con el broker MQTT local en Docker Compose.

```
docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":19,"hum":51}'
```

Deberás observar cómo se actualizan los indicadores de aguja. 

Para que se dibujen las lineas en la gráfica historica, necesitas enviar al menos un segundo dato. Puedes enviar de nuevo el mismo comando.

## Resultados

A continuación puede verse una vista previa del resultado de este flow.

![](https://github.com/hugoescalpelo/flow4-nodeRed-docker-compose/blob/main/imagenes/Screenshot%20from%202023-05-25%2001-12-03.png)

![](https://github.com/hugoescalpelo/flow4-nodeRed-docker-compose/blob/main/imagenes/Screenshot%20from%202023-05-25%2001-12-32.png)



## Evidencias

 [TikTok](https://www.tiktok.com/@hugoescalpelo/video/7236974959220755717)

## Problemas comunes

- Si usas un broker publico, usa la IP del broker ya que este puede cambiar en cualquier momento y NodeRed no actualiza las IPs luego de configurar un dominio. Para conocer la IP de un broker usa el comando ```nslookup [dominio_broker]```.
- Si la información recibida en el nodo MQTT no es detectada por el nodo JSON, asegurate de que es expresada como Sring.
- Si el nodo JSON marca error o caracter no esperado, asegurate de que el mensaje MQTT que enviaste describe correctamente un JSON. Ejemplo: `docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":23,"hum":50}'`

# Notas
- Este repositorio cuenta con las instrucciones para crear el flow pero también incluye el archivo JSON resultante, así que solo tienes que importarlo a nodeRed.

# Créditos

Desarrollado por Hugo Escalpelo
- [hugoescalpelo.com](https://hugoescalpelo.com/)
- [Página en Facebook](https://www.facebook.com/Hugo-Escalpelo-Profesional-337708683840136)
- [GitHub](https://github.com/hugoescalpelo)