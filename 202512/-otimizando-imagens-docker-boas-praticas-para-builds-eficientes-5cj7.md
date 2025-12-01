---
Title: # Otimizando Imagens Docker: Boas Práticas para Builds Eficientes
Description: 
Author: Victor Zarzar
Date: 2025-12-01T22:04:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>As imagens Docker são a base das aplicações containerizadas. No entanto, imagens grandes e ineficientes podem resultar em builds mais lentos, tempos maiores de implantação e aumento no uso de armazenamento. Otimizar imagens garante entregas mais rápidas, melhor desempenho e menor consumo de recursos.</p>

<h2>
  
  
  Efeitos de Imagens Grandes e Ineficientes na Implantação
</h2>

<p>Imagens pesadas não apenas consomem mais espaço em disco, mas também aumentam o tempo de transferência na rede, o que é crítico em pipelines CI/CD e implantações em nuvem. Imagens eficientes tornam as operações mais ágeis e reduzem custos operacionais.</p>

<h2>
  
  
  1. Uso de Imagens Base Slim
</h2>

<h3>
  
  
  Escolhendo Imagens Mínimas (Alpine, Variantes Slim)
</h3>

<p>Comece com uma imagem base mínima ou "slim" para reduzir o excesso de pacotes. Por exemplo, em vez de usar python:3.10, considere python:3.10-slim ou alpine. Imagens mínimas removem pacotes e bibliotecas desnecessárias, resultando em imagens menores e mais seguras.</p>

<h3>
  
  
  Reduzindo Dependências Desnecessárias
</h3>

<p>Instale apenas os pacotes e bibliotecas indispensáveis para sua aplicação. Evite incluir ferramentas de build ou arquivos de documentação na imagem final de runtime.</p>

<h3>
  
  
  Equilibrando Tamanho e Funcionalidade
</h3>

<p>Embora imagens slim sejam pequenas, algumas aplicações podem exigir bibliotecas específicas. Escolha uma imagem base que equilibre tamanho reduzido e funcionalidade necessária, evitando erros em tempo de execução.</p>

<h2>
  
  
  2. Multi-Stage Builds
</h2>

<h3>
  
  
  O Que São Multi-Stage Builds
</h3>

<p>Multi-stage builds permitem separar o ambiente de build do ambiente de runtime. Essa técnica possibilita incluir ferramentas de compilação em uma etapa e copiar apenas os artefatos finais para a imagem final, reduzindo drasticamente o tamanho.</p>

<h3>
  
  
  Separando Ambientes de Build e Runtime
</h3>

<p>No multi-stage:</p>

<p><strong>Stage 1:</strong> compila ou constrói a aplicação com todas as dependências necessárias.</p>

<p><strong>Stage 2:</strong> copia apenas o resultado final para uma imagem base menor, descartando ferramentas de build e arquivos intermediários.</p>

<h3>
  
  
  Exemplo prático (Node.js + pnpm):
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="w"> </span><span class="s">node:22-alpine</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="s">base</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="k">COPY</span><span class="s"> package*.json pnpm-lock.yaml* ./</span>

<span class="k">RUN </span>npm <span class="nb">install</span> <span class="nt">-g</span> pnpm <span class="se">\
</span>    <span class="o">&amp;&amp;</span> pnpm <span class="nb">install</span> <span class="nt">--frozen-lockfile</span>

<span class="k">EXPOSE</span><span class="s"> 3000</span>

<span class="k">CMD</span><span class="s"> ["pnpm", "run", "dev"]</span>
</code></pre>

</div>



<p>Este modelo reduz bastante o tamanho final, eliminando ferramentas de build e reaproveitando cache de dependências via pnpm.</p>

<h3>
  
  
  Exemplo Multi-Stage com Python Slim:
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">FROM</span><span class="s"> python:3.12-slim</span>

<span class="k">ENV</span><span class="s"> PYTHONPATH=/app</span>

<span class="k">WORKDIR</span><span class="s"> /app</span>

<span class="k">COPY</span><span class="s"> requirements.txt .</span>

<span class="k">RUN </span>pip <span class="nb">install</span> <span class="nt">--no-cache-dir</span> <span class="nt">--upgrade</span> <span class="nt">-r</span> requirements.txt

<span class="k">COPY</span><span class="s"> alembic.ini .</span>

<span class="k">COPY</span><span class="s"> ./entrypoint.sh .</span>

<span class="k">COPY</span><span class="s"> . .</span>

<span class="k">EXPOSE</span><span class="s"> 8000</span>

<span class="k">ENTRYPOINT</span><span class="s"> [ "./entrypoint.sh" ]</span>

<span class="k">CMD</span><span class="s"> ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]</span>
</code></pre>

</div>



<p>A imagem final permanece extremamente pequena, contendo apenas o runtime necessário.</p>

<h2>
  
  
  3. Como as Camadas Docker Afetam o Tamanho da Imagem
</h2>

<p>Cada instrução no Dockerfile (RUN, COPY, ADD) cria uma camada. Apesar do cache, muitas camadas pequenas podem aumentar o tamanho total da imagem.</p>

<p>Uma forma de otimizar é combinar comandos relacionados em um único RUN:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">RUN </span>apt-get update <span class="o">&amp;&amp;</span> <span class="se">\
</span>    apt-get <span class="nb">install</span> <span class="nt">-y</span> curl git <span class="o">&amp;&amp;</span> <span class="se">\
</span>    apt-get clean <span class="o">&amp;&amp;</span> <span class="se">\
</span>    <span class="nb">rm</span> <span class="nt">-rf</span> /var/lib/apt/lists/<span class="k">*</span>
</code></pre>

</div>



<p>Isso gera uma única camada, mantendo a imagem menor.</p>

<p>Além disso, sempre remova caches e arquivos temporários dentro do mesmo comando <code>RUN</code>. Caso contrário, eles permanecem nas camadas anteriores e aumentam o tamanho final.</p>

<h2>
  
  
  4. Otimização com .dockerignore
</h2>

<p>O arquivo <code>.dockerignore</code> impede que arquivos desnecessários sejam enviados ao contexto de build.</p>

<p><strong>Exemplo básico:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>node_modules
.next
.tests
__pycache__
__pytest__
builds/
venv
logs
</code></pre>

</div>



<p>Isso reduz drasticamente o contexto enviado para o Docker, acelerando o build e evitando camadas desnecessárias.</p>

<h2>
  
  
  5. Cache de Forma Eficiente
</h2>

<p>Podemos aproveitar o cache do Docker copiando primeiro os arquivos de dependências:</p>

<p><strong>Para Node:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">COPY</span><span class="s"> package.json pnpm-lock.yaml ./</span>
<span class="k">RUN </span>pnpm <span class="nb">install</span>
</code></pre>

</div>



<p><strong>Para Python:</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight docker"><code><span class="k">COPY</span><span class="s"> requirements.txt .</span>
<span class="k">RUN </span>pip <span class="nb">install</span> <span class="nt">-r</span> requirements.txt
</code></pre>

</div>



<p>As dependências só serão reprocessadas quando realmente mudarem.</p>

<h2>
  
  
  6. Boas Práticas Gerais
</h2>

<ul>
<li><p>Use tags como DOCKER_TAG=1.0.0 ou IMAGE_VERSION=1.0.0 para rastrear versões e evitar confusão entre builds</p></li>
<li><p>Faça auditorias periódicas com docker image ls e remova imagens antigas</p></li>
<li><p>Integre otimizações no pipeline CI/CD para garantir builds consistentes, menores e mais rápidos</p></li>
<li><p>Evite ferramentas de teste e documentação na imagem final de produção</p></li>
<li><p>Prefira imagens minimalistas sempre que possível</p></li>
</ul>

<h2>
  
  
  Conclusão
</h2>

<p>Adotar essas boas práticas eleva o desempenho das aplicações, reduz custos operacionais e garante imagens compactas e eficientes para ambientes modernos e escaláveis. Multi-stage builds, uso de imagens slim, limpeza de camadas e .dockerignore são pilares essenciais para criar imagens com alta performance e fácil manutenção.</p>

