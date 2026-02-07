---
Title: Publicación de paquetes npm turborepo, trusted publisher, pnpm
Description: 
Author: Joel Humberto Gómez Paredes
Date: 2026-02-07T22:03:49.000Z
Robots: noindex,nofollow
Template: index
---
<p>Hace una semana me di a la tarea de hacer un update a mi <a href="https://github.com/dezkareid/dezkareid" rel="noopener noreferrer">monorepo personal</a> (aún seguimos trabajando en ello).</p>

<p>Dentro de todas las tareas tuve que realizar la de publicar paquetes fue de las más complicadas y vengo a explicarles un poco del porque puede ser complejo.</p>

<h2>
  
  
  Old way
</h2>

<p>Mi proceso anterior era bastante simple:</p>

<ul>
<li>Entrar a <a href="https://npmjs.com/" rel="noopener noreferrer">npmjs</a>.</li>
<li>Generar un token de acceso.</li>
<li>Agregar el token a tu proceso de publicación.</li>
<li>Crear la env var de NPM_TOKEN y el GITHUB_TOKEN a semantic-release.</li>
</ul>

<p>Y listo, pero se cambiaron ciertas políticas de seguridad en npmjs que ahora sugiere el uso de <a href="https://docs.npmjs.com/trusted-publishers" rel="noopener noreferrer">trusted publishing</a> para la publicación usando <a href="https://openid.net/developers/how-connect-works/" rel="noopener noreferrer">OpenID Connect</a> en lugar de un token tradicional.</p>

<h2>
  
  
  Configuración de trusted publishing
</h2>

<p>Antes de crear la configuración del trusted publisher en npmjs, tu paquete debe existir en npmjs si o si.</p>

<p>En el caso de que vayas a publicar un paquete nuevo, lo que debes hacer es una publicación manual (<code>pnpm publish --tag dummy</code>) o usar una herramienta llamada <a href="https://github.com/azu/setup-npm-trusted-publish" rel="noopener noreferrer">setup-npm-trusted-publish</a>.</p>

<p>Ahora debes acceder a la sección de configuración de tu paquete en <code>https://www.npmjs.com/package/&lt;package-name/access</code> o <code>https://www.npmjs.com/package/&lt;scope&gt;/&lt;package-name/access</code>. Después debes agregar los datos de tu trusted publisher (github o gitlab).</p>

<p>Debe aparecerte algo similar a esto:<br>
<a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4y6cwq9pwq5gs26av9gp.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4y6cwq9pwq5gs26av9gp.png" alt="Seleccionando trusted publisher" width="800" height="272"></a></p>

<p>Una vez seleccionado solo hay que llenar datos:</p>

<ol>
<li>Organización o usuario</li>
<li>Repositorio</li>
<li>Nombre del archivo de tu workfflow (incluyendo extensión). <em>Nota: este debe estar dentro de tu folder de .github/workflows</em> .</li>
</ol>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frrzrwzbkhdr74cldgnjr.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frrzrwzbkhdr74cldgnjr.png" alt="Form de trusted publisher" width="800" height="582"></a></p>

<p>En mi caso fue:</p>

<ol>
<li>dezkareid</li>
<li>dezkareid</li>
<li>ci-packages.yml</li>
</ol>

<p>Listo ya la configuración esta completa, ahora hay que realizar la configuración de nuestro publisher en nuestro workflow.</p>
<h2>
  
  
  Configuración de trusted publishing en Github Action
</h2>

<p>Lo primero que necesitamos es configurar los permisos en nuestro job<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">permissions</span><span class="pi">:</span>
  <span class="na">id-token</span><span class="pi">:</span> <span class="s">write</span>
</code></pre>

</div>



<p>Estrictamente solo necesitamos el permiso de id-token para la conexión usando OpenID Connect y podríamos usar npm publish, pero como uso semantic-release para subir el tag a github y el changelog, entre otras cosas. Mi configuración es la siguiente:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">permissions</span><span class="pi">:</span>
  <span class="na">contents</span><span class="pi">:</span> <span class="s">write</span>
  <span class="na">issues</span><span class="pi">:</span> <span class="s">write</span>
  <span class="na">pull-requests</span><span class="pi">:</span> <span class="s">write</span>
  <span class="na">id-token</span><span class="pi">:</span> <span class="s">write</span>
</code></pre>

</div>



<p>Aquí hay un <a href="https://semantic-release.gitbook.io/semantic-release/recipes/ci-configurations/github-actions#github-workflows-release.yml-configuration-for-node-projects" rel="noopener noreferrer">ejemplo de un action</a> que usa esta configuración.</p>

<p>Mi configuración de semantic-release<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight json"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"plugins"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
    </span><span class="s2">"@semantic-release/commit-analyzer"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"@semantic-release/release-notes-generator"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"@semantic-release/changelog"</span><span class="p">,</span><span class="w">
    </span><span class="s2">"@semantic-release/npm"</span><span class="p">,</span><span class="w">
    </span><span class="p">[</span><span class="w">
      </span><span class="s2">"@semantic-release/git"</span><span class="p">,</span><span class="w">
      </span><span class="p">{</span><span class="w">
        </span><span class="nl">"assets"</span><span class="p">:</span><span class="w"> </span><span class="p">[</span><span class="w">
          </span><span class="s2">"CHANGELOG.md"</span><span class="w">
        </span><span class="p">]</span><span class="w">
      </span><span class="p">}</span><span class="w">
    </span><span class="p">],</span><span class="w">
    </span><span class="s2">"@semantic-release/github"</span><span class="w">
  </span><span class="p">]</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre>

</div>



<p>En esta parte tuve algunos problemas debido a mi arquitectura y a mi stack.</p>

<p>Yo uso una arquitectura de monorepo y no de polirepo, eso agrega complejidad ya que debes ubicarte especificamente en el proyecto que debes publicar. Ademas de que estoy usando pnpm entonces ocurrió algo raro que me pedía autenticarme. Al ser un monorepo uso workspaces y no de manejan de la misma manera usando npm que pnmp.</p>

<p>Entre los problemas que enfrente fueron los siguientes:</p>

<ol>
<li><p>Capa 8, no agregué correctamente el nombre del archivo de mi workflow, me di cuenta de esto porque al tratar de publicar el plugin de npm verifica si el OpenID connect esta bien configurado y me decía que no encontraba la configuración.</p></li>
<li><p>Tenía un comando release usando que era ejecutado directamente por <a href="https://turborepo.dev/" rel="noopener noreferrer">turborepo</a>, el problema es que este aplica un filtro de variables de entorno y debes indicar explicitamente cuales necesitas. Lo solucioné usando el folder con el cambio como working directory y ejecutando semantic-release directamente</p></li>
<li><p>El <a href="https://github.com/semantic-release/npm" rel="noopener noreferrer">plugin de semantic release de npm</a> no estaba funcionando, el problema no era el plugin sino mi stack. Al parecer había un detalle de como se manejaba la autenticación, la documentación de las herramientas por lo regular contempla que estes usando npm pero a bajo nivel puede ser que aunque tengas una herramienta que funcione de manera similar no siempre va a funcionar con tu stack. La solución fue instalar el plugin <a href="https://www.npmjs.com/package/@anolilab/semantic-release-pnpm" rel="noopener noreferrer">@anolilab/semantic-release-pnpm</a>. </p></li>
</ol>

<p>Mi configuración quedó de la siguiente manera<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="pi">{</span>
  <span class="s2">"</span><span class="s">plugins"</span><span class="pi">:</span> <span class="pi">[</span>
    <span class="s2">"</span><span class="s">@semantic-release/commit-analyzer"</span><span class="pi">,</span>
    <span class="s2">"</span><span class="s">@semantic-release/release-notes-generator"</span><span class="pi">,</span>
    <span class="s2">"</span><span class="s">@semantic-release/changelog"</span><span class="pi">,</span>
    <span class="s2">"</span><span class="s">@anolilab/semantic-release-pnpm"</span><span class="pi">,</span>
    <span class="pi">[</span>
      <span class="s2">"</span><span class="s">@semantic-release/git"</span><span class="pi">,</span>
      <span class="pi">{</span>
        <span class="s2">"</span><span class="s">assets"</span><span class="pi">:</span> <span class="pi">[</span>
          <span class="s2">"</span><span class="s">CHANGELOG.md"</span>
        <span class="pi">]</span>
      <span class="pi">}</span>
    <span class="pi">],</span>
    <span class="s2">"</span><span class="s">@semantic-release/github"</span>
  <span class="pi">],</span>
  <span class="s2">"</span><span class="s">extends"</span><span class="pi">:</span> <span class="s2">"</span><span class="s">semantic-release-monorepo"</span>
<span class="pi">}</span>
</code></pre>

</div>



<p>Y con esto ahora ya puedo publicar paquetes nuevamente usando mi stack preferido. Te dejo mi <a href="https://github.com/dezkareid/dezkareid/blob/main/.github/workflows/ci-packages.yml" rel="noopener noreferrer">workflow</a> para que puedas observarlo y si te sirve lo puedas replicar :D</p>

