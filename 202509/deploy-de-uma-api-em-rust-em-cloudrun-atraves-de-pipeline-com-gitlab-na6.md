---
Title: Deploy de uma api em rust em cloudrun através de pipeline com gitlab
Description: 
Author: Rodrigo Burgos
Date: 2025-09-08T21:21:10.000Z
Robots: noindex,nofollow
Template: index
---
<p>Post genuíno, em português e escrito por mim (gpt revisou hehehe). Tentarei ser o máximo objetivo possível mas sinta-se convidado a discutir abordagens ou tirar dúvidas por dm ou nos comentários.</p>

<ol>
<li>API</li>
</ol>

<p>A abordagem que utilizei para esta api é a de DDD (<em>domain drive design</em>). Não vou me demorar muito no desenvolvimento da api em sí, vou focar no <code>m̀ain.rs</code> e na árvore de arquivos.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>use test_api::infra::handler::health::get_health_handler;
use tokio;
use warp::Filter;

#[tokio::main]
async fn main() {
    // Define the route
    let get_health_route = warp::path("health")
        .and(warp::get())
        .and_then(get_health_handler);

    // Start the warp server
    warp::serve(get_health_route)
        .run(([0, 0, 0, 0], 8080))
        .await;
}

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>├── Cargo.toml
├── Dockerfile
├── REAME.md
├── src
│   ├── domain
│   │   ├── entities
│   │   └── use_case
│   ├── infra
│   │   └── handler
│   ├── lib.rs
│   ├── main.rs
│   └── use_case
│       └── get_health
└── target
    ├── CACHEDIR.TAG
    └── debug
        ├── build
        ├── deps
        ├── examples
        └── incremental

15 directories, 6 files

</code></pre>

</div>



<p>Pontos importantes para focar são os paramêtros do run do warp/serve:</p>

<p><code>.run(([0, 0, 0, 0], 8080))</code></p>

<p>Para que o container seja acessado externamente.</p>

<ol>
<li>Dockerfile
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code># Etapa 1: Builder com Rust + OpenSSL
FROM rust:latest AS builder

RUN apt-get update &amp;&amp; apt-get install -y pkg-config libssl-dev

WORKDIR /usr/src/test_api

# Copia arquivos do crate
COPY Cargo.toml ./
COPY src ./src

# Gera lockfile e baixa dependências
RUN cargo generate-lockfile
RUN cargo build --release

# Etapa 2: Imagem final com compatibilidade glibc
FROM debian:bookworm-slim

# Instala OpenSSL e dependências mínimas de runtime
RUN apt-get update &amp;&amp; apt-get install -y libssl3 ca-certificates &amp;&amp; \
    apt-get clean &amp;&amp; rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder /usr/src/test_api/target/release/test_api .

ENV PORT=8080
EXPOSE 8080

CMD ["./test_api"]
</code></pre>

</div>



<ol>
<li>Gitlab
</li>
</ol>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>stages:
    - deploy_cloud_run

deploy_cloud_run:
    stage: deploy_cloud_run
    tags: [temp]
    environment:
        name: dev
    rules:
        - if: '$CI_COMMIT_BRANCH == "dev" &amp;&amp; $CI_PIPELINE_SOURCE == "push"'
          changes:
              - **/*
          when: always
        - when: never
    before_script:
        - mkdir -p "$CI_PROJECT_DIR/.tmp/google-cloud-sdk"
        - curl -sSL "https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-438.0.0-linux-x86_64.tar.gz" | tar -xz -C "$CI_PROJECT_DIR/.tmp/google-cloud-sdk" --strip-components=1
        - export PATH="$CI_PROJECT_DIR/.tmp/google-cloud-sdk/bin:$PATH"
        - echo "$SERVICE_ACCOUNT_JSON" | base64 -d &gt; gcloud-key.json
        - gcloud auth activate-service-account --key-file=gcloud-key.json
        - export GCP_PROJECT_ID="seu-projeto"
        - gcloud config set project "$GCP_PROJECT_ID"
        - gcloud auth configure-docker us-central1-docker.pkg.dev --quiet
    script:
        - |
            IMAGE_TAG="$(git rev-parse --short HEAD)"
            IMAGE_URI="us-central1-docker.pkg.dev/seu-repo/api/test-api:${IMAGE_TAG}"

            docker build \
              -t "$IMAGE_URI" \
              -f test_api/Dockerfile \
              test_api

            docker push "$IMAGE_URI"

            gcloud run deploy api-dev \
              --image "$IMAGE_URI" \
              --platform managed \
              --region us-central1 \
              --allow-unauthenticated \
              --project "$GCP_PROJECT_ID" \
              --set-env-vars "ENVIRONMENT=staging"
    after_script:
        - rm -f gcloud-key.json
</code></pre>

</div>



<p>Permissões qua service account necessita para rodar esse job:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>gcloud projects add-iam-policy-binding tilt-network \ --member="serviceAccount:service-account@project.iam.gserviceaccount.com" \ --role="roles/run.admin"

gcloud projects add-iam-policy-binding tilt-network \ --member="serviceAccount:service-account@project.iam.gserviceaccount.com" \ --role="roles/viewer"

gcloud projects add-iam-policy-binding tilt-network \ --member="serviceAccount:service-account@project.iam.gserviceaccount.com" \ --role="roles/iam.serviceAccountUser"
</code></pre>

</div>



<p>O json da SA deve ser armazenado como segredo do Gitlab.</p>

<p>Dentro da sua gitlab dashboard, na sessão de jobs, deve existir um output como esse:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>Service [api-dev] revision [sua-api-00000-pez] has been deployed and is serving 100 percent of traffic.
Service URL: https://sua-api-hash-uc.a.run.app
</code></pre>

</div>



<p>Se tudo deu certo, utilizando curl para interagir com o endpoint health:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>curl https://sua-api-hash-uc.a.run.app/health
{"health_status":{"status":"healthy","message":"API is working fine"}}%
</code></pre>

</div>



<p>Repositório da api <a href="https://github.com/burgossrodrigo/test_api" rel="noopener noreferrer">aqui</a>.</p>

<p>Espero que, na eventualidade de você se encontrar travado com um procedimento como esse, ou parecido, essa postagem te ajuda a desimpedir (:</p>

