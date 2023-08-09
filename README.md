# flow5-clima-api-DC
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
6. Verifica que todos los nodos JSON estén configurados para siempre convertir a JSON.
7. Asegurate que los nodos function contienen el código correcto.

    Nodo function Temperatura

    ```
    msg.payload = msg.payload.temp;
    msg.topic = "Temperatura";
    return msg;
    ```
    Nodo function Humedad
    ```
    msg.payload = msg.payload.hum;
    msg.topic = "Humedad";
    return msg;
    ```
    Nodo function Temperatura API
    ```
    msg.payload = msg.payload.main.temp;
    msg.topic = "Temperatura";
    return msg;
    ```
    Nodo function Humedad API
    ```
    msg.payload = msg.payload.main.humidity;
    msg.topic = "Humedad";
    return msg;
    ```
8. Asegurate de configurar los nodos dashboard para que se representen en una pestaña y un grupo existente.
9. Verifica que el nodo **inject** esté lanzando un timestamp cada minuto.
10. Comprueba que tu broker mosquitto sea accesible desde el exterior del contenedor. La forma fácil de hacerlo, es verificar que el nodo MQTT del flow indique **conectado**.

### Instrucciónes de operación

1. Dirígete al dashboard en [localhost:1880/ui](http://locahost:1880/ui/)
2. Espera al menos 2 minutos despues de haber importado el flow y haber hecho clic en el botón **Deploy** para observar datos en el bloque API.
3. Envía al menos dos mensajes MQTT que incluyan un JSON con la temperatura y la humedad al tema `codigoIoT/mqtt/clima` de tu broker local. Ejemplo.

    ```
    docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":23,"hum":50}'
    ```

## Resultados

Cuando haya funcionado, verás los indicadores con valores y las gráficas indicando los cambios históricos.

![](https://github.com/hugoescalpelo/flow5-clima-api-docker-compose/blob/main/Imagenes/Screenshot%20from%202023-05-26%2003-00-43.png?raw=true)

![](https://github.com/hugoescalpelo/flow5-clima-api-docker-compose/blob/main/Imagenes/Screenshot%20from%202023-05-26%2001-19-39.png?raw=true)

**Consejo**: Intenta cambiar el rango de las gráficas historicas a 1 semana y deja el flow funcionando por varios días, observarás la dinamica de clima.

## Evidencias

[]()

## FAQ

- **P**. ¿Cómo agrego un volumen que contenga el archivo de configuración de mosquitto?. **R**. Realiza los siguientes pasos.

    - Detener el contenedor de Mosquitto 
    
        ```docker stop [id_contenedor]```
        
    - Eliminar el contenedor de Mosquitto
    
        ```docker rm [id_contenedor]```
        
    - Eliminar la imagen de Mosquitto

        ```
        docker images
        docker rmi [id_imagen]
        ```
    - Detener docker compose. Hay que entrar al directorio de compose

        ```docker compose stop```
        
    - Actualizar el archivo compose para que use el volumenes. 

    - Agrega tu archivo mosquitto.conf. Puedes tomar como ejemplo el archivo de configuración de mi repositorio [servidor-IoT-basico-docker-compose](https://github.com/hugoescalpelo/servidor-IoT-basico-docker-compose)

    - Levantar docker compose

        ```docker compose up -d```

## Problemas comúnes

- Si usas un broker publico, usa la IP del broker ya que este puede cambiar en cualquier momento y NodeRed no actualiza las IPs luego de configurar un dominio. Para conocer la IP de un broker usa el comando ```nslookup [dominio_broker]```.
- Si la información recibida en el nodo MQTT no es detectada por el nodo JSON, asegurate de que es expresada como Sring.
- Si el nodo JSON marca error o caracter no esperado, asegurate de que el mensaje MQTT que enviaste describe correctamente un JSON. Ejemplo: `docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":23,"hum":50}'`

# Créditos

Desarrollado por Hugo Escalpelo y copiado por Carlos Rosales ;)
- [hugoescalpelo.com](https://hugoescalpelo.com/)
- [Página en Facebook](https://www.facebook.com/Hugo-Escalpelo-Profesional-337708683840136)
- [GitHub](https://github.com/hugoescalpelo)