---
Title: Safeinstall: Um aliado no combate a ataques de Supply chain
Description: 
Author: Ronaldo Modesto
Date: 2026-02-19T22:10:25.000Z
Robots: noindex,nofollow
Template: index
---
<p>Olá, pessoal!</p>

<p>Se você é desenvolvedor ou, pelo menos, envolvido com a área tech, tenho certeza de que já ouviu falar de ataques de supply chain, ou ataques de cadeia de suprimentos.</p>

<p>Esse tipo de ataque tem se tornado cada vez mais frequente, ainda mais com IAs gerando cada vez mais código sem supervisão. Com isso em mente, criei uma ferramenta que visa impedir que dependências comprometidas sejam instaladas em seu projeto, reduzindo o risco de um ataque bem-sucedido de cadeia de suprimentos.</p>

<p>Chega mais para conhecer! E, claro, todo feedback é super bem-vindo. 🙂</p>

<p>Ha e claro, se quiser saber mais sobre o que é supply chain atack, dá uma conferida aqui <a href="https://dev.to/r9n/como-funciona-supply-chain-attack-ptbr-31i0">O que é Suply Chain Attack</a></p>

<h2>
  
  
  A Crise de Segurança na Cadeia de Suprimentos Open Source
</h2>

<p>Os ataques de supply chain (cadeia de suprimentos) têm aumentado de forma alarmante nos últimos anos. Dados recentes revelam um cenário preocupante:</p>

<ul>
<li>
<strong>Malware em código aberto subiu 156%</strong> de um ano para outro em 2024, com mais de 704 mil pacotes maliciosos identificados desde 2019.</li>
<li>Em 2025, houve um <strong>aumento de 73% em pacotes open source maliciosos</strong> detectados em comparação com 2024.</li>
<li>Entre 2020 e 2023, as ameaças em repositórios open source <strong>cresceram 1.300%</strong>.</li>
<li>O custo global anual projetado de ataques de supply chain deve alcançar <strong>US$ 60 bilhões em 2025</strong>.</li>
<li>No ecossistema <strong>npm</strong>, a atividade maliciosa mais do que dobrou em 2025, representando quase <strong>90% de todo o malware open source</strong> detectado.</li>
<li>Terceiros comprometidos passaram a representar <strong>30% de todas as violações de dados</strong> em 2025.</li>
</ul>

<p>Instalar dependências sem verificação tornou-se uma operação de risco. Mesmo pacotes populares podem conter vulnerabilidades conhecidas ou, em cenários mais graves, serem comprometidos por atacantes.</p>




<h2>
  
  
  O que é o SafeInstall ✅?
</h2>

<p>O <strong>SafeInstall</strong> é um wrapper de segurança para comandos de instalação de pacotes que consulta a API do <a href="https://osv.dev" rel="noopener noreferrer">OSV (Open Source Vulnerabilities)</a> para identificar vulnerabilidades conhecidas <strong>antes</strong> de permitir que a instalação seja concluída. Assim, ele ajuda a reduzir ataques de supply chain alertando o desenvolvedor sobre pacotes vulneráveis antes que eles entrem no projeto.</p>

<p><strong>Repositório da ferramenta</strong> 🚀: <a href="https://github.com/R9n/safeinstall" rel="noopener noreferrer">Github</a> </p>

<h3>
  
  
  Benefícios Principais
</h3>

<ol>
<li>
<strong>Verificação antes da instalação</strong> — Nada é instalado sem checagem de vulnerabilidades.</li>
<li>
<strong>Integração com OSV</strong> — Usa dados agregados do GitHub Advisory, PyPI Advisory, CVE/NVD, RustSec, Go Vulnerability Database e outros.</li>
<li>
<strong>Suporte a múltiplos ecossistemas</strong> — npm, pip, Go, Cargo, RubyGems, Packagist, Pub.</li>
<li>
<strong>Instalações em lote</strong> — Protege comandos como <code>npm install</code> e <code>pip install -r requirements.txt</code>.</li>
<li>
<strong>Controle nas suas mãos</strong> — Permite confirmar ou cancelar instalações com risco identificado.</li>
</ol>

<p>A seguir temos mostro exemplos de como a ferramenta funciona e como ela pode auxiliar na proteção desse tipo de ataque.</p>

<ul>
<li>Bloqueando instalação de biblioteca especificando uma versão</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6hf7e8bzuir5321o1qx.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe6hf7e8bzuir5321o1qx.png" alt="blocking specific version" width="668" height="330"></a></p>

<ul>
<li>Bloqueando instalação de biblioteca utilizando instalação em massa, por exemplo npm i</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvktyjhq1tw1wyi5z0b3f.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fvktyjhq1tw1wyi5z0b3f.png" alt="blocking batch instalation" width="800" height="408"></a></p>

<ul>
<li>Permitindo instalação de bibliotecas mesmo com vulnerabilidades(ambientes de CI/CD por exemplo)</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fht0ibcg8yobckrg74vsh.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fht0ibcg8yobckrg74vsh.png" alt="allowing vulnerable lib" width="800" height="506"></a></p>

<ul>
<li>Quando uma versão não é especificada, a ferramenta tenta identificar automaticamente a última versão e então analisa para ver se alguma vulnerabilidade é encontrada</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2h25xs929wky2f99itmw.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2h25xs929wky2f99itmw.png" alt="identifying lib version automatically" width="800" height="448"></a></p>

<ul>
<li>Exemplo golang para instalação vulnerável vs instalação de última versão</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3oxcpcjzrg35kfd6ivdl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3oxcpcjzrg35kfd6ivdl.png" alt="example golang vulnerable and not vulnerable lib" width="800" height="295"></a></p>

<ul>
<li>Exemplo de biblioteca python</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwk1astjk3xlxp5ypvid2.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwk1astjk3xlxp5ypvid2.png" alt="python example pip lib manager" width="791" height="317"></a></p>

<ul>
<li>Exemplo de biblioteca sem vulnerabilidades em lotes</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frfbk5156zofon5qozjhq.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frfbk5156zofon5qozjhq.png" alt="batch isntall with no vulnerability" width="800" height="453"></a></p>

<ul>
<li>Exemplo de biblioteca sem vulnerabilidade</li>
</ul>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0mv0iz9deic0vbbrwgq7.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F0mv0iz9deic0vbbrwgq7.png" alt="example of lib with no vulnerability" width="789" height="447"></a></p>




<h2>
  
  
  Como Instalar 💥
</h2>

<h3>
  
  
  Pré-requisitos
</h3>

<ul>
<li>
<strong>Go 1.25.7+</strong> — Necessário para compilar o SafeInstall e para o osv-scanner.</li>
<li>
<strong>osv-scanner</strong> — Necessário para varreduras em lote. É instalado automaticamente pelos scripts de instalação.</li>
</ul>

<p>Para instalar o osv-scanner manualmente (se necessário):<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go <span class="nb">install </span>github.com/google/osv-scanner/v2/cmd/osv-scanner@v2
</code></pre>

</div>



<p>Garanta que <code>$(go env GOPATH)/bin</code> esteja no seu PATH.</p>

<h3>
  
  
  Instalação por plataforma
</h3>

<p><strong>Windows (PowerShell):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight powershell"><code><span class="o">.</span><span class="n">\scripts\install.ps1</span><span class="w">
</span></code></pre>

</div>



<p><strong>Windows (Prompt de Comando):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight batchfile"><code><span class="kd">scripts</span>\install.bat
</code></pre>

</div>



<p><strong>Linux / macOS:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="nb">chmod</span> +x scripts/install.sh
./scripts/install.sh
</code></pre>

</div>



<p><strong>Compilação manual:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>go build <span class="nt">-o</span> safeinstall <span class="nb">.</span>
</code></pre>

</div>






<h2>
  
  
  Comandos e Uso
</h2>

<h3>
  
  
  Sintaxe básica
</h3>

<p><strong>Sintaxe nova (recomendada):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>safeinstall [-y|--yes] &lt;ecosistema&gt; "&lt;pacote&gt;"
</code></pre>

</div>



<p><strong>Sintaxe legada (comando completo):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>safeinstall [-y|--yes] "&lt;comando de instalação&gt;"
</code></pre>

</div>



<h3>
  
  
  Instalação de pacote único
</h3>

<p>Ao especificar um pacote com versão, o SafeInstall:</p>

<ol>
<li>Consulta a API do OSV em busca de vulnerabilidades conhecidas</li>
<li>Aplica o algoritmo IsVulnerable para saber se sua versão exata é afetada</li>
<li>Se houver vulnerabilidade: exibe CVE/GHSA em vermelho e pede confirmação</li>
<li>Se estiver seguro: exibe mensagem em verde e prossegue automaticamente</li>
</ol>

<p><strong>Exemplos (sintaxe nova):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>safeinstall npm <span class="s2">"express"</span>
safeinstall npm <span class="s2">"express@4.17.1"</span>
safeinstall pip <span class="s2">"requests==2.25.1"</span>
safeinstall pip <span class="s2">"jinja2"</span>
safeinstall go <span class="s2">"golang.org/x/crypto@v0.1.0"</span>
</code></pre>

</div>



<p><strong>Exemplos (sintaxe legada):</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>safeinstall <span class="s2">"npm i express"</span>
safeinstall <span class="s2">"pip install -r requirements.txt"</span>
</code></pre>

</div>



<h3>
  
  
  Instalação em lote (npm i, pip install -r, etc.)
</h3>

<p>Para comandos que não especificam um único pacote (ex.: <code>npm install</code>, <code>npm ci</code>, <code>pip install -r requirements.txt</code>):</p>

<ol>
<li>O <strong>osv-scanner</strong> roda <strong>antes</strong> da instalação, analisando manifests e lockfiles</li>
<li>Se forem encontradas vulnerabilidades e você não passou <code>-y</code>/<code>--yes</code>, o SafeInstall pede confirmação</li>
<li>Se você recusar, a <strong>instalação é cancelada</strong> — pacotes vulneráveis não são instalados</li>
<li>Se não houver vulnerabilidades (ou você aceitar o risco), o comando de instalação é executado</li>
<li>Após a instalação, o osv-scanner roda novamente para verificar as dependências instaladas</li>
</ol>

<p><strong>Exemplos:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>safeinstall <span class="s2">"npm install"</span>
safeinstall <span class="s2">"npm ci"</span>
safeinstall <span class="s2">"pip install -r requirements.txt"</span>
</code></pre>

</div>



<h3>
  
  
  Modo CI/CD
</h3>

<p>Use <code>-y</code> ou <code>--yes</code> para aceitar automaticamente os riscos em ambientes automatizados:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>safeinstall <span class="nt">-y</span> <span class="s2">"npm install express@4.17.1"</span>
</code></pre>

</div>



<h3>
  
  
  Desinstalação
</h3>

<p>Para remover o SafeInstall por completo:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>safeinstall <span class="nt">--uninstall</span>
<span class="c"># ou</span>
safeinstall <span class="nt">-u</span>
<span class="c"># ou</span>
safeinstall uninstall
</code></pre>

</div>



<p>Ou use os scripts de desinstalação:</p>

<ul>
<li>
<strong>Windows (PowerShell):</strong> <code>.\scripts\uninstall.ps1</code>
</li>
<li>
<strong>Windows (CMD):</strong> <code>scripts\uninstall.bat</code>
</li>
<li>
<strong>Linux/macOS:</strong> <code>./scripts/uninstall.sh</code>
</li>
</ul>




<h2>
  
  
  Ecossistemas Suportados
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>Ecossistema</th>
<th>Gerenciador de Pacotes</th>
<th>Exemplo</th>
</tr>
</thead>
<tbody>
<tr>
<td>npm</td>
<td>npm, yarn, pnpm</td>
<td><code>safeinstall npm "express@4.17.1"</code></td>
</tr>
<tr>
<td>PyPI</td>
<td>pip, pip3, poetry</td>
<td><code>safeinstall pip "requests==2.25.1"</code></td>
</tr>
<tr>
<td>Go</td>
<td>go get</td>
<td><code>safeinstall go "golang.org/x/crypto@v0.1.0"</code></td>
</tr>
<tr>
<td>crates.io</td>
<td>cargo</td>
<td><code>safeinstall "cargo add serde@1.0"</code></td>
</tr>
<tr>
<td>RubyGems</td>
<td>gem</td>
<td><code>safeinstall "gem install rails:7.0"</code></td>
</tr>
<tr>
<td>Packagist</td>
<td>composer</td>
<td>Pacotes PHP</td>
</tr>
<tr>
<td>Pub</td>
<td>pub</td>
<td>Pacotes Dart</td>
</tr>
</tbody>
</table></div>




<h2>
  
  
  Como o SafeInstall Reduz o Risco de Supply Chain
</h2>

<ol>
<li>
<strong>Bloqueio proativo</strong> — Evita instalações de pacotes conhecidamente vulneráveis ou maliciosos.</li>
<li>
<strong>Uso de dados confiáveis</strong> — OSV agrega bases como GitHub Advisory, PYSEC, CVE e RustSec.</li>
<li>
<strong>Avaliação precisa</strong> — Usa o algoritmo oficial do OSV (IncludedInVersions, IncludedInRanges, BeforeLimits).</li>
<li>
<strong>Dupla verificação em lote</strong> — Analisa antes e depois da instalação em cenários como <code>npm install</code> e <code>pip install -r</code>.</li>
<li>
<strong>Decisão consciente</strong> — O desenvolvedor escolhe se aceita o risco em casos específicos.</li>
<li>
<strong>Integração simples</strong> — Substitui chamadas diretas aos gerenciadores de pacotes com um comando único.</li>
</ol>




<h2>
  
  
  Requisitos
</h2>

<ul>
<li>Conexão com a internet para consultas à API</li>
</ul>

<h2>
  
  
  - Go e osv-scanner no PATH (para instalações em lote)
</h2>




<p><strong>Fontes:</strong></p>

<ul>
<li>[1] Supply Chain Attack Statistics 2025: Costs &amp; Defenses — DeepStrike. <a href="https://deepstrike.io/blog/supply-chain-attack-statistics-2025" rel="noopener noreferrer">https://deepstrike.io/blog/supply-chain-attack-statistics-2025</a>
</li>
<li>[2] The State of the Software Supply Chain 2025 — JFrog. <a href="https://jfrog.com/blog/state-of-software-supply-chain-security-2025" rel="noopener noreferrer">https://jfrog.com/blog/state-of-software-supply-chain-security-2025</a>
</li>
<li>[3] ReversingLabs 2026 Software Supply Chain Security Report — 73% aumento em pacotes open source maliciosos. <a href="https://www.reversinglabs.com/press-releases/reversinglabs-2026-software-supply-chain-security-report-identifies-73-increase-in-malicious-open-source-packages" rel="noopener noreferrer">https://www.reversinglabs.com/press-releases/reversinglabs-2026-software-supply-chain-security-report-identifies-73-increase-in-malicious-open-source-packages</a>
</li>
<li>[4] Sonatype's 10th Annual State of the Software Supply Chain Report — 156% de aumento em malware open source. <a href="https://www.globenewswire.com/news-release/2024/10/10/2961239/0/en/Sonatype-s-10th-Annual-State-of-the-Software-Supply-Chain-Report-Reveals-156-Surge-in-Open-Source-Malware.html" rel="noopener noreferrer">https://www.globenewswire.com/news-release/2024/10/10/2961239/0/en/Sonatype-s-10th-Annual-State-of-the-Software-Supply-Chain-Report-Reveals-156-Surge-in-Open-Source-Malware.html</a>
</li>
</ul>

