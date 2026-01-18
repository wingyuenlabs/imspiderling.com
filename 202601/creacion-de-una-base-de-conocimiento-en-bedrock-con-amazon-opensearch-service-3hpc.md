---
Title: Creacion de una base de conocimiento en Bedrock con Amazon OpenSearch Service.
Description: 
Author: Francy Hernandez Vega
Date: 2026-01-18T21:40:46.000Z
Robots: noindex,nofollow
Template: index
---
<p>Amazon Bedrock es el servicio de AWS que revolucionó la forma en que podemos acceder, interactuar y probar  modelos fundacionales (FM) de las principales empresas de IA.</p>

<p>Amazon Bedrock  nos permite conectarnos  con los principales modelos funcionales (FM) y la creación de bases de conocimiento que conectan las aplicaciones a fuentes de datos que contienen la información en diversos recursos como textos, imágenes, videos o recursos externos como base para la generación de los datos que respondan a casos de uso como creación de asistentes virtuales, flujo entre agentes o creación de nuevo contenido, logrando experimentar de forma rápida las ideas de los equipos de trabajo. </p>

<p>Conceptos previos:</p>

<ul>
<li>
<strong>Embedding</strong>: En este proceso el modelo convierte el texto o datos en una serie de números (vector) que permite a Bedrock entender el significado de los datos.</li>
<li>
<strong>Almacenamiento</strong>: Estos vectores se guardan en una base de datos vectorial como Amazon OpenSearch Service. Allí los datos se organizan para crear un índice que permita su búsqueda y recuperación.</li>
<li>
<strong>Búsqueda</strong>: Al realizar una consulta o pregunta sobre estos datos, se convierte en un nuevo vector que se usará para encontrar en la base de datos los vectores más cercanos a dicha consulta.</li>
<li>
<strong>Creción</strong>: El modelo seleccionado en Amazon Bedrock utilizará los datos recuperados por los vectores para generar una respuesta precisa. </li>
</ul>

<p>En el siguiente Demo probaremos las ventajas de Amazon Bedrock Knowledge Bases integrado al servicio de Amazon Open Search como base de datos vectorial para la recuperación de información a partir de una fuente de texto.</p>

<p><strong>Paso 1:</strong> Crear el bucket en S3 que contendrá el archivo .pdf con la información que se tomará como base para responder a las consultas.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn1ym8lcgutmc938xsnvp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn1ym8lcgutmc938xsnvp.png" alt=" " width="800" height="203"></a></p>

<p><strong>Paso 2:</strong> Crear Base de Conocimiento con almacenamiento de vector.<br>
Ingresamos al servicio de Amazon Bedrock, en el menú izquierdo en la sección Build seleccionados Knowledge Bases y en la opción crear seleccionamos <strong>Knowledge Bases with vector store</strong>.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fipefi0c428rzei3z1m6i.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fipefi0c428rzei3z1m6i.png" alt=" " width="669" height="322"></a></p>

<p><strong>Paso 3:</strong> Configurar almacenamiento y procesamiento.<br>
Seleccionamos el modelo embebido para convertir los datos a vectores. Adicional seleccionar el tipo de almacenamiento en este caso Amazon OpenSearch Serverless.</p>

<p><strong>Recomendación</strong>: Consultar previamente la información sobre el costo de Amazon OpenSearch</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffye9j80pxu1ogzpjigl1.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ffye9j80pxu1ogzpjigl1.png" alt=" " width="651" height="302"></a></p>

<p>Una vez terminado se mostrará la creacion de la base de conocimiento, la seleccionamos y damos clic en Sync para sincronizar la fuente de datos S3 con Amazon OpenSearch:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ha4ywj3ma8cg55ze6yq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F6ha4ywj3ma8cg55ze6yq.png" alt=" " width="800" height="232"></a></p>

<p>Ahora ingresamos al servicio de  Amazon Opensearch: en el menú izquierdo seleccionamos Servelresss y damos clic en Dashboard.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmu940yiqtxda4bb2epn3.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fmu940yiqtxda4bb2epn3.png" alt=" " width="800" height="320"></a></p>

<p>Allí vemos la colección creada para la base de conocimientos que usará Bedrock. Ingresamos al link del tablero donde crearemos el índice que identificara la partición de la información.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faaptwghn226pg52wvmyt.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faaptwghn226pg52wvmyt.png" alt=" " width="800" height="292"></a></p>

<p>Este enlace nos direccionara al tablero de OpenSearch, damos clic en la opción de “Explore on my own”. Para ver nuestros datos en el menú izquierdo damos clic en “Discover”.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8f7qygef31wylamkclam.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8f7qygef31wylamkclam.png" alt=" " width="800" height="360"></a></p>

<p>Creamos el nuevo indice.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcy1yhj8omh0yprlpksu7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcy1yhj8omh0yprlpksu7.png" alt=" " width="800" height="280"></a></p>

<p>Nuevamente en la sección de Discover veremos  el detalle de los vectores. Allí el documento fuente almacenado en S3, fue dividido en porciones de texto(chunk) que luego fueron convertidos a su representación en vectores creados por el modelo embebido en Bedrock. </p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqr31s0xtmxu2lx7ufom.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feqr31s0xtmxu2lx7ufom.png" alt=" " width="800" height="389"></a></p>

<p><strong>¡Hora de Probar!</strong></p>

<p>Regresamos a Amazon Bedrock, seleccionamos la base de conocimiento creada y damos clic en “Test Knowledge Base”</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faumqd48jy8oqpdyjr47d.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faumqd48jy8oqpdyjr47d.png" alt=" " width="800" height="329"></a></p>

<p>Seleccionamos el modelo fundacional que tomará la información para generar las respuestas. En la sección “Test” ingresamos la primera pregunta que enviaremos al modelo.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frkgaxtc0drlesl5ae3gu.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frkgaxtc0drlesl5ae3gu.png" alt=" " width="603" height="268"></a></p>

<p>En la respuesta se mostrará la información encontrada, la referencia de la fuente consultada, las secciones o chunk de donde fueron extraídos los datos y el detalle de cada uno de ellos.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftcj2ovh8jabghn7tn358.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftcj2ovh8jabghn7tn358.png" alt=" " width="800" height="327"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe1weo54vv5hfzj1pz4qo.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe1weo54vv5hfzj1pz4qo.png" alt=" " width="800" height="377"></a></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk6aq6z0mbq355mfbfy6f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fk6aq6z0mbq355mfbfy6f.png" alt=" " width="800" height="395"></a></p>

<p><strong>Recordatorio:</strong> Una vez terminado nuestro laboratorio eliminar los recursos creados: la Base de comocimientos en Bedrock y la coleccion de Amazon OpenSearch Service.</p>

