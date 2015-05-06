# ejemplo para crear imágenes docker con spring-boot

este no es ni un tutorial, apenas unas breves explicaciones de como crear una aplicación spring-boot que genere una imágen docker y cómo ejecutarla.

##1. Instalar docker
ver https://docs.docker.com/installation/ que está super bien explicado para todos los sistemas operativos.

concretamente, en macosx:

1. prerequisitos
 - [virtualbox](https://www.virtualbox.org/)
 - jdk 1.7+
 - maven 3+
2. instalar docker y boot2docker
```sh
$ brew install docker
$ brew install boot2docker
```
3. iniciarlos
```sh
$ boot2docker init
$ boot2docker up
```
4. editar el ~/.bash_profile para añadir al final
```
export DOCKER_HOST=tcp://192.168.59.103:2376
export DOCKER_CERT_PATH=/Users/javier/.boot2docker/certs/boot2docker-vm
export DOCKER_TLS_VERIFY=1
$(boot2docker shellinit 2> /dev/null)
```
donde 192.168.59.103 es la IP que habéis visto en la inicialización
5. probar que funciona
```sh
docker run -i -t ubuntu /bin/bash
```

##2. clonar este proyecto
```sh
$ git clone https://github.com/jantoniucci/spring-boot-docker.git
```
##3. entenderlo
en ```src/main/docker/Dockerfile``` tenemos la definición para crear nuestro contenedor:
```
FROM java:7
VOLUME /tmp
ADD gs-spring-boot-docker-0.1.0.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
como se puede leer en el fichero, se indica la imágen de la que se parte, dónde se montará el volumen de datos, se añade el jar que hemos generado y se define el comando a ejecutar. más info en https://docs.docker.com/reference/builder/.

en el ```pom.xml``` definimos el plugin automágico de la gente de spotify que creará la imágen:
```xml
<properties>
      <docker.image.prefix>springio</docker.image.prefix>
</properties>
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.2.3</version>
            <configuration>
                <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                <dockerDirectory>src/main/docker</dockerDirectory>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>
    </plugins>
</build>
```
la configuración es bastante auto-explicativa.

##4. ejecutar la imagen
para ejecutarla primero debemos construirla:
```sh
$ mvn package docker:build
```
ahora si se puede ejecutar haciendo:
```sh
$ docker run -p 8080:8080 -t jantoniucci/spring-boot-docker
```
para ver el "servicio" spring-boot en un navegador hacemos:
```sh
$ open http://192.168.59.103:8080
```
o metemos esa url en un navegador, donde la ip es la que corresponde a la que os haya informado docker al inicializar.

para ver los procesos docker corriendo:
```sh
$ docker ps
```
y para pararlo:
```sh
$ docker stop 54e2398383
```
utilizando el identificador que hayais encontrado con el ps

##5. publicar la imagen
si queréis publicar a un repo de docker, deberéis personalizar en el pom.xml la propiedad ```docker.image.prefix``` para poner vuestro usuario. 

luego ejecutar:
```sh
$ docker push vuestrousuario/spring-boot-docker
```

##6. conclusiones
* gracias a los chicos de spotify la creación de la imágen queda super mavenizada
* los parámetros dependientes de entorno se pueden pasar como parámetros de la ejecución
* spring-boot + docker = microservicios ágiles

te gustan estos temas? Entonces tienes que conocer lo que hacemos en [Adesis Netlife](http://www.adesis.com).

te gusta este tutorial? Márcalo como favorito y twitealo al mundo!

## Contacto
 * [javier.antoniucci@gmail.com](mailto:javier.antoniucci@gmail.com)
 * http://www.adesis.com
 
## Contribuciones
 * Preguntas, erratas, solicitudes: escribir en la sección Issues del repo
 * Correcciones, mejoras, añadidos : enviar pull-requests
 * Propuestas, colaboraciones en otros tutoriales, etc: javier.antoniucci@gmail.com

## Licencia
[Beerware Software Licence](http://en.wikipedia.org/wiki/Beerware). Beerware es un término de licencia de software otorgado bajo términos muy libres. Provee al usuario final el derecho a un programa particular (y hacer lo que quiera con el código fuente). Si el usuario considera el software útil, se le exhorta a comprarle al autor una cerveza "para devolver el favor".
