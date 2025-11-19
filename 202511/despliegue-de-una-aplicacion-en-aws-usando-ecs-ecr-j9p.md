---
Title: Despliegue de una aplicación en AWS usando ECS + ECR
Description: 
Author: Robert Orozco 
Date: 2025-11-19T21:40:16.000Z
Robots: noindex,nofollow
Template: index
---
<p>En este post te cuento cómo desplegué una aplicación en AWS usando Elastic Container Service (ECS) y Elastic Container Registry (ECR).</p>

<p>👉 La idea es: contenerizamos la app, subimos la imagen, armamos un clúster ECS y la aplicación queda corriendo en ECS (Con EC2)</p>

<p><strong>ARQUITECTURA</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2ibnsp5askbmyw07zs0f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2ibnsp5askbmyw07zs0f.png" alt=" " width="800" height="779"></a></p>

<p><strong>🐳 1. Preparar la instancia EC2 para construir la imagen</strong></p>

<p>Primero lanzamos una EC2 con Amazon Linux 2023 para preparar la imagen que luego enviaremos a ECR.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft72p04o3dpn9dtoq6lex.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ft72p04o3dpn9dtoq6lex.png" alt=" " width="800" height="213"></a></p>

<p><strong>🔧 Instalar Docker y Git</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>sudo dnf update -y
sudo yum install -y docker
sudo systemctl enable --now docker
sudo systemctl status docker
sudo yum install -y git
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F60h10r4g3e4xqgs3g4ic.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F60h10r4g3e4xqgs3g4ic.png" alt=" " width="800" height="310"></a></p>

<p>📥 Clonar el repositorio y construye la imagen.</p>

<p>Clona la carpeta 4 del repositorio: <a href="https://github.com/NotHarshhaa/DevOps-Projects/tree/master/DevOps-Project-04" rel="noopener noreferrer">https://github.com/NotHarshhaa/DevOps-Projects/tree/master/DevOps-Project-04</a><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>git clone &lt;URL-del-proyecto&gt;
cd &lt;directorio&gt;
docker build -t hello-world-django-app:version-1 .
</code></pre>

</div>



<p>Verifica que la imagen existe:</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsozu7s3z319mj6896ap6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsozu7s3z319mj6896ap6.png" alt=" " width="800" height="80"></a></p>

<p><strong>📦 2. Crear el repositorio en ECR</strong></p>

<p>En la consola de AWS:</p>

<p>ECR → Repositories → Create repository</p>

<p>Ponle un nombre como aplicaciones ✔.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F51f81bz4zrslohch63qf.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F51f81bz4zrslohch63qf.png" alt=" " width="800" height="263"></a></p>

<p>Este repo será donde guardaremos nuestras imágenes Docker para que ECS pueda consumirlas.</p>

<p><strong>🔐 3. Rol IAM para permitir que EC2 interactúe con ECR</strong></p>

<p>1️⃣ Crear el rol</p>

<p>IAM → Roles → Create Role → EC2<br>
Agrega la politica:<br>
AmazonEC2ContainerRegistryPowerUser (push + pull)<br>
Nombre sugerido: ec2-ecr-role</p>

<p>2️⃣ Asignar el rol a tu instancia EC2</p>

<p>EC2 → Instance → Actions → Security → Modify IAM role<br>
Selecciona ec2-ecr-role.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Focil8ezu53mkjvbb4w93.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Focil8ezu53mkjvbb4w93.png" alt=" " width="800" height="307"></a></p>

<p>*<em>🔑 4. Login en ECR + Push de la imagen<br>
*</em><br>
En la EC2 define estas variables:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
AWS_REGION=us-east-2
ECR_URI=${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

</code></pre>

</div>



<p>🔐 Loguearte en ECR<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>aws ecr get-login-password --region $AWS_REGION \
  | docker login --username AWS --password-stdin $ECR_URI
</code></pre>

</div>



<p>🏷 Etiquetar la imagen<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>docker tag hello-world-django-app:version-1 $ECR_URI/aplicaciones:version-1
</code></pre>

</div>



<p>🚀 Subir la imagen (push)<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>docker push $ECR_URI/aplicaciones:version-1
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F35z6qps48fhpfdt9oh1v.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F35z6qps48fhpfdt9oh1v.png" alt=" " width="800" height="235"></a></p>

<p>¡Imagen lista en ECR! 🐳📦</p>

<p>☁️ 5. Crear el clúster ECS</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72iopxj4706d35gux6jb.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F72iopxj4706d35gux6jb.png" alt=" " width="800" height="218"></a></p>

<p><strong>🔧 Configuración del clúster</strong><br>
Selecciona Fargate e instancias EC2 autoadministradas<br>
Crea un Auto Scaling Group “bajo demanda” (modo prueba: 1 instancia)<br>
Selecciona el rol básico de ECS<br>
Elige dos subredes mínimo (ej: us-east-2a y us-east-2b)<br>
Crea un SG llamado ecs-sg</p>

<p>📑 6. Definición de tarea (Task Definition)</p>

<p>Aquí le decimos a ECS qué contenedor correr, cuánta memoria, puertos, etc.</p>

<p>⚙️ Configuración principal</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdeqpezgtqujfsloe1fe.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftdeqpezgtqujfsloe1fe.png" alt=" " width="800" height="304"></a></p>

<p>Nombre: app-task<br>
Infraestructura: EC2<br>
Modo de red: bridge<br>
CPU: 0.5 vCPU<br>
Memoria: 1 GB<br>
Rol de ejecución: el de ECS</p>

<p>📦 Contenedor<br>
Nombre: django-app<br>
Image URI: selecciona la imagen que subiste a ECR 🐳<br>
Puertos: 8000:8000</p>

<p><strong>🎯 7. Crear el servicio ECS</strong></p>

<p>Para que la app quede corriendo de forma continua:</p>

<p>ECS → Cluster → Create Service<br>
Familia de tareas: app-task<br>
Launch type: EC2<br>
Número de tareas: 1</p>

<p><strong>🌐 8. Probar la app en el navegador</strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsym9sdcpcsnoq26iaqqv.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fsym9sdcpcsnoq26iaqqv.png" alt=" " width="800" height="386"></a></p>

<p>Con esta arquitectura se ha desplegada una aplicacion mediante el uso de ECR y ECS para orquestar contenedores.</p>

