#Configurar Local Registry
Arrancamos el registry local:
```
docker run -d -p 5000:5000 --restart=always --name registry -v C:/Users/Eugenio/Documents/imagenes:/var/lib/registry -v C:/Users/Eugenio/Documents/imagenes/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v C:/Users/Eugenio/Documents/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/euge.pem -e REGISTRY_HTTP_TLS_KEY=/certs/euge.key registry:2
```

Hacemos login:
```
docker login www.gz.com:5000
```

Vamos a usar este builder:
´´´
pack inspect-builder cnbs/sample-builder:bionic 


Remote
------

Stack: io.buildpacks.samples.stacks.bionic

Lifecycle:
  Version: 0.4.0
  Buildpack API: 0.2
  Platform API: 0.1

Run Images:
  cnbs/sample-stack-run:bionic

Buildpacks:
  ID                                         VERSION
  io.buildpacks.samples.java-maven           0.0.1
  io.buildpacks.samples.kotlin-gradle        0.0.1
  io.buildpacks.samples.ruby-bundler         0.0.1

Detection Order:
  Group #1:
    io.buildpacks.samples.java-maven@0.0.1
  Group #2:
    io.buildpacks.samples.kotlin-gradle@0.0.1
  Group #3:
    io.buildpacks.samples.ruby-bundler@0.0.1

Local
-----

Not present
```

Podemos ver que run imagen usa. Vamos a descargar los Builders y la run image:  
´´´
docker pull cnbs/sample-builder:bionic 
docker pull cnbs/sample-stack-run:bionic
```

Les ponemos un tag:  
```
docker tag cnbs/sample-builder:bionic www.gz.com:5000/cnbs/sample-builder:bionic 
docker tag cnbs/sample-stack-run:bionic www.gz.com:5000/cnbs/sample-stack-run:bionic 
```

Vamos a subirlos a nuestro registry local:  
```
docker push www.gz.com:5000/cnbs/sample-builder:bionic 
docker push www.gz.com:5000/cnbs/sample-stack-run:bionic 
```
# El Builder
Inspeccionamos el builder:  
```
pack inspect-builder www.gz.com:5000/cnbs/sample-builder:bionic 
```

Podemos especificar el builder por defecto:  
```
pack set-default-builder www.gz.com:5000/cnbs/sample-builder:bionic 
```

Podemsos configurar un mirror para nuestro run image:
```
pack set-run-image-mirrors cnbs/sample-stack-run:bionic --mirror www.gz.com:5000/cnbs/sample-stack-run:bionic 
```

Al especificar el mirror, si hicieramos:
```
pack inspect-builder www.gz.com:5000/cnbs/sample-builder:bionic
```

Veriamos:
```
Run Images:
  www.gz.com:5000/cnbs/sample-stack-run:bionic (user-configured)
  cnbs/sample-stack-run:bionic

```
  
# Crear una imagen
Podriamos hacer simplemente:  

```
pack build aplicacion
```

Seria equivalente a:
```
pack build aplicacion --builder www.gz.com:5000/cnbs/sample-builder:bionic
```

Podemos tambien publicar la imagen con pack:
```
pack build aplicacion --builder www.gz.com:5000/cnbs/sample-builder:bionic --publish
```

Podemos ver ahora la imagen resultante:
```
docker inspect-image aplicacion --bom | jq .Remote
```