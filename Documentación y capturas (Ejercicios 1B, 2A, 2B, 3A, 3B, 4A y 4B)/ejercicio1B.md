# EJERCICIO 1B. Blog Jekyll en un bucket AWS S3

**Alumno:** RubéN Marcos Cuadrado
**Bucket:** `iaw-actividad21-1b-cuadradoruben`  
**Región:** REGION  
**Endpoint:** `http://iaw-actividad21-1b-apellidonombre.s3-website-REGION.amazonaws.com`

> Para el nombre del bucket utilizo `1b` en minúsculas, ya que AWS S3 no admite letras mayúsculas en los nombres de los buckets.

## 1. Preparar el sitio

Antes de publicar el blog en AWS, tengo que adaptar la configuración de Jekyll al nuevo entorno.

En GitHub Pages, un sitio de proyecto puede estar alojado dentro de una ruta como `/miblog`, por lo que `baseurl` debe contener esa ruta. En cambio, al utilizar Amazon S3, el sitio se publica directamente en la raíz del alojamiento, por lo que dejo `baseurl` vacío.

Primero abro el archivo `_config.yml` del proyecto y modifico los siguientes parámetros:
previamente hacemos una copia de seguridad por sia caso

```yaml
baseurl: ""
url: "http://iaw-actividad21-1b-cuadradoruben.s3-website-us-east-1.amazonaws.com"
```

Guardo los cambios y abro una terminal para acceder al directorio del proyecto:

```bash
cd ~/miblog
```

A continuación, genero la versión estática del sitio con Jekyll:

```bash
bundle exec jekyll build
```

Este comando genera los archivos del sitio web dentro del directorio `_site`, que será el contenido que posteriormente subiré al bucket de S3.

Compruebo que se han generado los dos archivos principales:

```bash
ls _site/index.html _site/404.html
```

Los archivos que necesito son:

- `index.html`: será la página principal del blog.
- `404.html`: será la página que se mostrará cuando se acceda a una dirección que no exista.

**Captura 1:** ejecución de la compilación y la comprobación de los archivos `index.html` y `404.html`.

--- 
![1](/miblog/Documentación/images/Captura1.png)


## 2. Crear y configurar el bucket en Amazon S3

Una vez preparado el sitio, entro en la consola web de AWS para crear el bucket que utilizaré como alojamiento.

### 2.1. Crear el bucket

Sigo estos pasos desde la consola de administración de AWS:

1. Accedo a la consola de AWS.
2. En el buscador de servicios escribo **S3** y entro en Amazon S3.
3. En el apartado de buckets, pulso **Crear bucket**.
4. En el campo del nombre introduzco:

   `iaw-actividad21-1b-cuadradoruben`

5. Selecciono la misma región que estoy utilizando en el resto de la práctica.
6. Mantengo el tipo de bucket y las opciones de configuración predeterminadas que no sea necesario modificar.
7. Reviso la configuración y pulso **Crear bucket**.

Una vez creado, compruebo que aparece correctamente en el listado de buckets.
![2](/miblog/Documentación/images/Captura2nombredelbuckets3.png)

### 2.2. Desactivar el bloqueo del acceso público

Por defecto, AWS bloquea el acceso público a los buckets. Como quiero utilizar S3 para alojar una web accesible desde Internet, tengo que modificar esta configuración.

Para ello:

1. Entro en el bucket que acabo de crear.
2. Abro la pestaña **Permisos**.
3. Busco el apartado **Bloquear el acceso público (configuración del bucket)**.
4. Pulso **Editar**.
5. Desmarco la opción **Bloquear todo el acceso público**.
6. AWS muestra un aviso indicando que el contenido podrá quedar accesible públicamente.
7. Confirmo el aviso y guardo los cambios.

Con esto permito que el bucket pueda tener objetos accesibles públicamente. Más adelante estableceré los permisos concretos mediante una política del bucket.

**Captura 3:** mostrar la configuración del bloqueo del acceso público desactivada.

![3](/miblog/Documentación/images/Captura3accesopúblicoparaestebucket.png)



### 2.3. Activar el alojamiento web estático

Ahora tengo que indicar a S3 que quiero utilizar el bucket para servir una página web estática.

Para hacerlo:

1. Entro en la pestaña **Propiedades** del bucket.
2. Bajo hasta el apartado **Alojamiento de sitios web estáticos**.
3. Pulso **Editar**.
4. Selecciono la opción **Habilitar**.
5. En el tipo de alojamiento selecciono **Alojar un sitio web estático**.
6. En el campo del documento de índice introduzco:

   `index.html`

7. En el campo del documento de error introduzco:

   `404.html`

8. Guardo los cambios.

Una vez guardada la configuración, AWS muestra los datos del alojamiento y genera un endpoint desde el que se podrá visitar el blog.

La dirección tendrá este formato:

```text
http://iaw-actividad21-1b-apellidonombre.s3-website-REGION.amazonaws.com
```

Sustituyo `REGION` por el identificador real de la región seleccionada al crear el bucket.

**Captura 4:** alojamiento web estático habilitado, los documentos configurados y el endpoint generado por AWS.

![4](/miblog/Documentación/images/Captura4Alojamientositiowebestaticoactivado.png)

### 2.4. Configurar la política del bucket

El siguiente paso es permitir que los visitantes puedan leer los archivos del blog.

Para ello:

1. Entro de nuevo en la pestaña **Permisos**.
2. Busco el apartado **Política del bucket**.
3. Pulso **Editar**.
4. Introduzco la siguiente política JSON, utilizando el nombre exacto de mi bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "LecturaPublica",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::iaw-actividad21-1b-apellidonombre/*"
  }]
}
```

5. Compruebo que el nombre del bucket coincide con el que he creado.
6. Guardo los cambios.

Esta política permite que cualquier usuario (`Principal: "*"`) pueda leer los objetos almacenados en el bucket mediante la acción `s3:GetObject`.

Los permisos son únicamente de lectura. Los visitantes no pueden subir, modificar ni eliminar archivos mediante esta política.

**Captura 5:** mostrar la política JSON guardada correctamente en el apartado de permisos.

![5](/miblog/Documentación/images/Captura5Politicabucketguardada.png)

---

## 3. Primera subida de los archivos a S3 desde la consola

Con el bucket preparado, ya puedo subir los archivos que ha generado Jekyll.

Para hacerlo desde la consola web:

1. Entro en Amazon S3.
2. Abro el bucket `iaw-actividad21-1b-cuadradoruben`.
3. Entro en la pestaña **Objetos**.
4. Pulso el botón **Cargar**.
5. Selecciono **Añadir archivos** o **Añadir carpeta**, según corresponda.
6. Desde el explorador de archivos, accedo al directorio `~/miblog/_site`.
7. Selecciono el contenido generado dentro de `_site`, es decir, los archivos y carpetas que contiene.
8. Compruebo que los elementos aparecen en la lista de archivos pendientes de carga.
9. Pulso **Cargar** y espero a que finalice el proceso.

Es importante subir el contenido de `_site` y no la carpeta `_site` como un directorio adicional. De esta forma, `index.html` queda directamente en la raíz del bucket y S3 puede encontrarlo como documento principal.

Cuando termina la carga, vuelvo a la pestaña **Objetos** y compruebo que aparecen los archivos y directorios del blog.

**Captura 6:** mostrar el listado de objetos después de completar la subida.

![6](/miblog/Documentación/images/cargadobucket.png)

### 3.1. Comprobar que el blog funciona

Una vez subidos los archivos, abro el endpoint que me ha proporcionado AWS en el navegador.

Voy a realizar cuatro comprobaciones para asegurarme de que el sitio se ha publicado correctamente.

**Primera comprobación: página principal**

Abro la dirección del endpoint y compruebo que aparece la portada del blog. También reviso que el contenido y los enlaces se muestran correctamente.

**Captura 7:** mostrar la página principal del blog publicada en S3.

![7](/miblog/Documentación/images/Paginaprincipalweb.png)

**Segunda comprobación: una entrada con imagen**

Accedo a una publicación que tenga una imagen y compruebo que tanto el texto como la imagen se cargan correctamente.

**Captura 8:** mostrar una entrada del blog que incluya una imagen.

![8](/miblog/Documentación/images/Captura8Entradaconimagenens3.png)


**Tercera comprobación: una página interna**

Accedo a otra página del sitio utilizando su enlace correspondiente. Compruebo que la navegación funciona y que se puede acceder a su contenido sin problemas.

**Captura 9:** mostrar una página interna del blog.

![9](/miblog/Documentación/images/Páginainternablogs3.png)

**Cuarta comprobación: página de error 404**

Por último, introduzco manualmente una dirección que no exista dentro del dominio del sitio.

Compruebo que AWS muestra el documento `404.html` que he configurado anteriormente, en lugar de una página genérica de error.

**Captura 10:** mostrar la página personalizada `404.html` al acceder a una dirección inexistente.

![10](/miblog/Documentación/images/paginanoexistente404.png)


---

## 4. Conclusiones

Con esta práctica he podido publicar un blog creado con Jekyll utilizando Amazon S3 como servicio de alojamiento web estático.

Durante el proceso he trabajado con la creación y configuración de buckets, la gestión de permisos públicos, las políticas de acceso y la activación del alojamiento web estático.

Una de las cosas que he comprobado es que Amazon S3 no ejecuta Jekyll ni genera las páginas web. La compilación se realiza previamente en local y S3 se encarga de almacenar y servir los archivos que se han generado.

También he comprobado que la configuración de `baseurl` depende del entorno de publicación. Para S3 lo he dejado vacío porque el sitio se sirve desde la raíz del bucket, mientras que en GitHub Pages puede ser necesario utilizar `/miblog` como ruta base.

Por último, el endpoint de alojamiento web estático de S3 utiliza HTTP. Para ofrecer una conexión HTTPS sería necesario añadir un servicio adicional como Amazon CloudFront.

En general, esta práctica me ha servido para conocer el funcionamiento del alojamiento web estático en AWS y trabajar con una alternativa a la publicación tradicional de páginas web desde un servidor propio.