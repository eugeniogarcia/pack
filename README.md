#Configurar Local Registry
Arrancamos el registry local:
```
docker run -d -p 5000:5000 --restart=always --name registry -v C:/Users/Eugenio/Documents/imagenes:/var/lib/registry -v C:/Users/Eugenio/Documents/imagenes/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v C:/Users/Eugenio/Documents/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/euge.pem -e REGISTRY_HTTP_TLS_KEY=/certs/euge.key registry:2
```

Hacemos login:
```
docker login www.gz.com:5000
```

descargamos los Builders y la run image:  
´´´
docker pull cloudfoundry/cnb:bionic 
docker pull cloudfoundry/run:base-cnb
```

Les ponemos un tag:  
```
docker tag cloudfoundry/cnb:bionic www.gz.com:5000/cloudfoundry/cnb:bionic 
docker tag cloudfoundry/run:base-cnb www.gz.com:5000/cloudfoundry/run:base-cnb 
```

Vamos a subirlos a nuestro registry local:  
```
docker push www.gz.com:5000/cloudfoundry/cnb:bionic 
docker push www.gz.com:5000/cloudfoundry/run:base-cnb 
```
# El Builder
Inspeccionamos el builder:  
```
pack inspect-builder cloudfoundry/cnb:bionic
pack inspect-builder www.gz.com:5000/cloudfoundry/cnb:bionic
```

Podemos especificar el builder por defecto:  
```
pack set-default-builder www.gz.com:5000/cloudfoundry/cnb:bionic
```

Podemsos configurar un mirror para nuestro run image:
```
pack set-run-image-mirrors cloudfoundry/run:base-cnb --mirror www.gz.com:5000/cloudfoundry/run:base-cnb 
```

Si ahora inspeccionamos nuestro builder:
´´´
pack inspect-builder www.gz.com:5000/cloudfoundry/cnb:bionic
```

Podemos observar:  
```
Local
-----

Description: Ubuntu bionic base image with buildpacks for Java, NodeJS and Golang

Stack: io.buildpacks.stacks.bionic

Lifecycle:
  Version: 0.4.0
  Buildpack API: 0.2
  Platform API: 0.1

Run Images:
  www.gz.com:5000/cloudfoundry/run:base-cnb (user-configured)
  cloudfoundry/run:base-cnb
  
...
```

# Crear una imagen
Podriamos hacer simplemente:  

```
pack build aplicacion
```

Seria equivalente a:
```
pack build aplicacion --builder www.gz.com:5000/cloudfoundry/cnb:bionic 
```

Podemos tambien publicar la imagen con pack:
```
pack build aplicacion --builder www.gz.com:5000/cloudfoundry/cnb:bionic www.gz.com:5000/app:latest -publish
```








docker inspect-image aplicacion --bom | jq .Remote
