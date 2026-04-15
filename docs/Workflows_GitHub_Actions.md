# 🐳 Padrão de Workflows Docker no GitHub Actions

## 🎯 Objetivo

Este documento padroniza o uso dos workflows de build e publish de imagens Docker no GitHub Actions para os projetos da esteira.

A ideia é simples:

> 📘 Este material serve como referência oficial para copy/paste e padronização dos workflows Docker no GitHub Actions.


- evitar reinventar pipeline por repositório
- manter governança de versionamento e publicação
- facilitar copy/paste para novos projetos
- separar claramente build contínuo, release estável e pre-release
- documentar o Template B para monorepo com múltiplas imagens e bundle offline

---

## 🧩 Visão geral dos templates


## 🔧 Convenção final adotada

A convenção oficial ficou assim:

- Git tags **sem** prefixo `v`
- imagens Docker **sem** prefixo `v`
- `main` publica `edge`
- release estável publica `stable` + versões
- pre-release publica apenas a tag literal
- `latest` não deve ser usado

Exemplos oficiais:
- release estável: `1.4.2`
- pre-release: `1.4.2-rc.1`

---


Temos dois modelos oficiais:

### 🅰️ Template A — Serviço único
Usado quando o repositório gera **uma única imagem Docker**.

Exemplos:
- API Auth
- API Integrator
- API Management
- qualquer microserviço isolado

### 🅱️ Template B — Monorepo com múltiplas imagens + bundle offline
Usado quando o repositório gera **várias imagens Docker** e também precisa montar um **bundle offline**.

Exemplo:
- solução completa com `auth`, `integrator`, `management`, `licensing-client`, `app-intellistor`

---

# 🏷️ Regras de versionamento

## 🌿 Branch `main`
Quando houver `push` na `main`, o workflow publica:

- uma tag imutável por commit: `sha-...`
- uma tag contínua de integração: `edge`

Exemplo:
```bash
git push origin main
```

Resultado esperado:
- `ghcr.io/<owner>/<repo>:sha-xxxxxxx`
- `ghcr.io/<owner>/<repo>:edge`

No Template B, por serviço:
- `ghcr.io/<owner>/<repo>:auth-edge`
- `ghcr.io/<owner>/<repo>:integrator-edge`
- `ghcr.io/<owner>/<repo>:management-edge`
- etc.

---

## ✅ Release estável
Quando houver publicação de uma tag SemVer estável, o workflow publica:

Padrão oficial adotado: **sem prefixo `v`**.

- `stable`
- versão completa
- major.minor
- major

Exemplo:
```bash
git tag 1.4.2
git push origin 1.4.2
```

Resultado esperado no Template A:
- `ghcr.io/<owner>/<repo>:stable`
- `ghcr.io/<owner>/<repo>:1.4.2`
- `ghcr.io/<owner>/<repo>:1.4`
- `ghcr.io/<owner>/<repo>:1`

Resultado esperado no Template B, por serviço:
- `ghcr.io/<owner>/<repo>:auth-stable`
- `ghcr.io/<owner>/<repo>:auth-1.4.2`
- `ghcr.io/<owner>/<repo>:auth-1.4`
- `ghcr.io/<owner>/<repo>:auth-1`

---

## 🧪 Pre-release
Quando houver uma tag SemVer com sufixo, o workflow publica apenas a tag literal.

Exemplo:
```bash
git tag 1.4.2-rc.1
git push origin 1.4.2-rc.1
```

Resultado esperado no Template A:
- `ghcr.io/<owner>/<repo>:1.4.2-rc.1`

Resultado esperado no Template B:
- `ghcr.io/<owner>/<repo>:auth-1.4.2-rc.1`
- `ghcr.io/<owner>/<repo>:integrator-1.4.2-rc.1`
- etc.

Importante:
- pre-release **não** publica `stable`
- pre-release **não** sobrescreve linha oficial de release

---

# 🚫 Por que não usar `latest`

O padrão oficial **não usa `latest`**.

Motivo:
- `latest` gera ambiguidade
- dificulta troubleshooting
- confunde integração contínua com release oficial
- pode causar pull de imagem errada em ambientes diferentes

Em vez disso, usamos:
- `edge` para integração contínua na `main`
- `stable` para a última release oficial
- versão explícita para rastreabilidade
- tag literal para pre-release

---

# 🅰️ Template A — Serviço único

## 📌 Quando usar
Use este template quando o repositório publicar apenas uma imagem Docker.

## 📋 Pode fazer copy/paste?
Sim.

Você pode copiar e colar esse template em qualquer projeto novo **desde que**:
- o branch principal seja `main`
- o registry seja `ghcr.io`
- o build use Dockerfile padrão
- o contexto seja simples
- o projeto precise publicar uma única imagem

## 🛠️ Quando precisa ajustar
Ajuste o template se houver diferença em:
- branch principal
- `context`
- `file` do Dockerfile
- registry
- secrets
- build args
- necessidade de testes antes do push
- necessidade de mono-arch em vez de multi-arch

## 📦 Exemplo oficial — Template A

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
    tags:
      - '*.*.*'
      - '*.*.*-*'
  workflow_dispatch:

concurrency:
  group: build-and-push-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    name: Build and Push Multi-Arch Image
    runs-on: ubuntu-latest

    env:
      DOCKER_BUILDKIT: 1

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          flavor: |
            latest=false
          tags: |
            type=sha
            type=raw,value=edge,enable=${{ github.ref == 'refs/heads/main' }}
            type=raw,value=stable,enable=${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') }}
            type=semver,pattern={{version}},enable=${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') }}
            type=semver,pattern={{major}}.{{minor}},enable=${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') }}
            type=semver,pattern={{major}},enable=${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') }}
            type=ref,event=tag,enable=${{ startsWith(github.ref, 'refs/tags/') && contains(github.ref_name, '-') }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          platforms: linux/amd64,linux/arm64
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: false
          sbom: false
```

---

# 🅱️ Template B — Monorepo com múltiplas imagens + bundle offline

## 📌 Quando usar
Use este template quando o repositório:
- gera várias imagens Docker
- possui múltiplos serviços no mesmo código
- precisa montar um bundle offline para instalação/entrega

## 🔒 O que foi preservado no modelo ajustado
O Template B ajustado:
- manteve os dois jobs
- manteve o build de múltiplas imagens
- manteve o bundle offline
- manteve o `docker save | gzip`
- manteve a montagem da pasta `bundle/`
- manteve a validação da estrutura final
- manteve o upload do artifact

Ou seja:
- o core funcional foi preservado
- só recebeu melhorias de padrão, governança e versionamento

## 🔄 O que mudou no padrão
Antes, o modelo antigo usava:
- `auth-latest`
- `integrator-latest`
- `management-latest`
- etc.

Agora, o padrão oficial usa:
- `auth-edge` na `main`
- `auth-stable` na release estável
- `auth-1.4.2` em release versionada
- `auth-1.4.2-rc.1` em pre-release

Isso melhora rastreabilidade e reduz ambiguidade.

Padrão final adotado:
- Git tags sem `v`
- imagens Docker sem `v`
- Template A e Template B seguem a mesma convenção

## 📦 Exemplo oficial — Template B

```yaml
name: Build and publish Docker images

on:
  push:
    branches: [ main ]
    tags:
      - '*.*.*'
      - '*.*.*-*'
  workflow_dispatch:

concurrency:
  group: build-and-push-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read
  packages: write

env:
  PLATFORMS: ${{ startsWith(github.ref, 'refs/tags/') && 'linux/amd64,linux/arm64' || 'linux/amd64' }}

jobs:
  build-and-push:
    name: Build and Push Multi-Service Images
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
      is_tag: ${{ steps.version.outputs.is_tag }}
      is_prerelease: ${{ steps.version.outputs.is_prerelease }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Determine version and release type
        id: version
        run: |
          if [[ "${GITHUB_REF}" == refs/tags/* ]]; then
            VERSION="${GITHUB_REF_NAME}"
            echo "version=${VERSION}" >> "$GITHUB_OUTPUT"
            echo "is_tag=true" >> "$GITHUB_OUTPUT"

            if [[ "${GITHUB_REF_NAME}" == *-* ]]; then
              echo "is_prerelease=true" >> "$GITHUB_OUTPUT"
            else
              echo "is_prerelease=false" >> "$GITHUB_OUTPUT"
            fi
          else
            echo "version=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"
            echo "is_tag=false" >> "$GITHUB_OUTPUT"
            echo "is_prerelease=false" >> "$GITHUB_OUTPUT"
          fi

      - name: Set up QEMU
        if: ${{ startsWith(github.ref, 'refs/tags/') }}
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push auth
        uses: docker/build-push-action@v6
        with:
          context: ./auth
          file: ./auth/Dockerfile
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:auth-${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || steps.version.outputs.version }}
            ${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') && format('ghcr.io/{0}:auth-stable', github.repository) || '' }}
          platforms: ${{ env.PLATFORMS }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: false
          sbom: false

      - name: Build and push licensing-client
        uses: docker/build-push-action@v6
        with:
          context: ./licensing-client
          file: ./licensing-client/Dockerfile
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:licensing-client-${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || steps.version.outputs.version }}
            ${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') && format('ghcr.io/{0}:licensing-client-stable', github.repository) || '' }}
          platforms: ${{ env.PLATFORMS }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: false
          sbom: false

      - name: Build and push integrator
        uses: docker/build-push-action@v6
        with:
          context: ./integrator
          file: ./integrator/Dockerfile
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:integrator-${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || steps.version.outputs.version }}
            ${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') && format('ghcr.io/{0}:integrator-stable', github.repository) || '' }}
          platforms: ${{ env.PLATFORMS }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: false
          sbom: false

      - name: Build and push management
        uses: docker/build-push-action@v6
        with:
          context: ./management
          file: ./management/Dockerfile
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:management-${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || steps.version.outputs.version }}
            ${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') && format('ghcr.io/{0}:management-stable', github.repository) || '' }}
          platforms: ${{ env.PLATFORMS }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: false
          sbom: false

      - name: Build and push app-intellistor
        uses: docker/build-push-action@v6
        with:
          context: ./app-intellistor/intellistor-platform
          file: ./app-intellistor/intellistor-platform/Dockerfile
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:app-intellistor-${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || steps.version.outputs.version }}
            ${{ startsWith(github.ref, 'refs/tags/') && !contains(github.ref_name, '-') && format('ghcr.io/{0}:app-intellistor-stable', github.repository) || '' }}
          platforms: ${{ env.PLATFORMS }}
          build-args: |
            NEXT_PUBLIC_URL_LICENSE=${{ vars.NEXT_PUBLIC_URL_LICENSE }}
            NEXT_PUBLIC_URL_AUTH=${{ vars.NEXT_PUBLIC_URL_AUTH }}
            NEXT_PUBLIC_URL_INTEGRATOR=${{ vars.NEXT_PUBLIC_URL_INTEGRATOR }}
            NEXT_PUBLIC_URL_MANAGEMENT=${{ vars.NEXT_PUBLIC_URL_MANAGEMENT }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: false
          sbom: false

  bundle-offline:
    name: Build Offline Bundle
    runs-on: ubuntu-latest
    needs: build-and-push
    if: ${{ startsWith(github.ref, 'refs/tags/') || github.event_name == 'workflow_dispatch' }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Pull all images (amd64 only for offline bundle)
        env:
          VERSION: ${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || needs.build-and-push.outputs.version }}
          REPO: ghcr.io/${{ github.repository }}
        run: |
          docker pull --platform linux/amd64 "${REPO}:auth-${VERSION}"
          docker pull --platform linux/amd64 "${REPO}:licensing-client-${VERSION}"
          docker pull --platform linux/amd64 "${REPO}:integrator-${VERSION}"
          docker pull --platform linux/amd64 "${REPO}:management-${VERSION}"
          docker pull --platform linux/amd64 "${REPO}:app-intellistor-${VERSION}"
          docker pull mysql:8.0
          docker pull nginx:alpine

      - name: Save and compress images
        env:
          VERSION: ${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || needs.build-and-push.outputs.version }}
          REPO: ghcr.io/${{ github.repository }}
        run: |
          docker save \
  "${REPO}:auth-${VERSION}" \
  "${REPO}:licensing-client-${VERSION}" \
  "${REPO}:integrator-${VERSION}" \
  "${REPO}:management-${VERSION}" \
  "${REPO}:app-intellistor-${VERSION}" \
  mysql:8.0 \
  nginx:alpine \
| gzip > intellistor-solution-bundle-${VERSION}.tar.gz

      - name: Prepare bundle staging directory
        env:
          VERSION: ${{ startsWith(github.ref, 'refs/heads/main') && 'edge' || needs.build-and-push.outputs.version }}
        run: |
          mkdir -p bundle

          cp compose.yml bundle/
          cp compose.poc.yml bundle/
          cp compose.external-db.yml bundle/
          cp compose.images.yml bundle/
          cp nginx.conf bundle/

          [ -f .env.common.example ] && cp .env.common.example bundle/ || true
          [ -f .env.poc.example ] && cp .env.poc.example bundle/ || true
          [ -f .env.external-db.example ] && cp .env.external-db.example bundle/ || true

          [ -d scripts ] && cp -r scripts/ bundle/ || true

          SERVICES=(auth integrator licensing-client management)
          RSYNC_EXCLUDES=(
            --exclude='node_modules/'
            --exclude='.next/'
            --exclude='.venv/'
            --exclude='dist/'
            --exclude='build/'
            --exclude='__pycache__/'
            --exclude='*.pyc'
            --exclude='.pytest_cache/'
            --exclude='*.egg-info/'
            --exclude='.pnpm-store/'
          )

          for svc in "${SERVICES[@]}"; do
            rsync -a "${RSYNC_EXCLUDES[@]}" "${svc}/" "bundle/${svc}/"
          done

          rsync -a "${RSYNC_EXCLUDES[@]}" "app-intellistor/" "bundle/app-intellistor/"

          mkdir -p bundle/data
          for svc in "${SERVICES[@]}"; do
            if [ -d "data/${svc}" ]; then
              cp -r "data/${svc}" "bundle/data/"
            fi
          done

          if [ -d "data/app-intellistor" ]; then
            cp -r "data/app-intellistor" "bundle/data/"
          fi

          [ -d initdb ] && cp -r initdb/ bundle/ || true

          mkdir -p bundle/logs
          if [ -d logs ]; then
            cp -r logs/* bundle/logs/ 2>/dev/null || true
          fi
          for svc in "${SERVICES[@]}" app-intellistor; do
            mkdir -p "bundle/logs/${svc}"
          done

          mv "intellistor-solution-bundle-${VERSION}.tar.gz" bundle/

      - name: Validate bundle structure
        run: |
          echo "=== Bundle directory tree (depth 3) ==="
          find bundle -maxdepth 3 -type d -print
          echo ""
          echo "=== Bundle files (depth 4, first 100) ==="
          find bundle -maxdepth 4 -type f -print | head -100

          FAIL=0
          for f in \
  bundle/compose.yml \
  bundle/compose.poc.yml \
  bundle/compose.external-db.yml \
  bundle/compose.images.yml \
  bundle/nginx.conf \
  bundle/.env.common.example \
  bundle/.env.poc.example \
  bundle/.env.external-db.example \
  bundle/data/auth/certs/ed25519_public.pem \
  bundle/data/integrator/certs/ed25519_public.pem \
  bundle/data/licensing-client/certs/ed25519_public.pem \
  bundle/data/management/certs/ed25519_public.pem \
  bundle/scripts/install-offline.sh \
  bundle/scripts/install-offline.ps1; do
            if [ ! -e "$f" ]; then
              echo "ERROR: Missing required bundle file: $f" >&2
              FAIL=1
            fi
          done

          if [ -d initdb ] && [ ! -e bundle/initdb/01-init.sql ]; then
            echo "ERROR: Missing required bundle file: bundle/initdb/01-init.sql" >&2
            FAIL=1
          fi

          if [ "$FAIL" -ne 0 ]; then
            echo "Bundle validation failed!" >&2
            exit 1
          fi

          echo "Bundle validation passed."

      - name: Upload intellistor-solution-bundle artifact
        uses: actions/upload-artifact@v4
        with:
          name: intellistor-solution-bundle
          path: bundle/
          retention-days: 30
```

---

# 🧪 Comandos prontos para o dev

## 🚀 Build contínuo na `main`
```bash
git add .
git commit -m "feat: ajuste no serviço"
git push origin main
```

---

## 🏁 Criar release estável
```bash
git add .
git commit -m "release: versão 1.4.2"
git push origin main

git tag 1.4.2
git push origin 1.4.2
```


---

## 🧬 Criar pre-release
```bash
git add .
git commit -m "release: rc 1.4.2-rc.1"
git push origin main

git tag 1.4.2-rc.1
git push origin 1.4.2-rc.1
```


---

## 👀 Ver tags locais
```bash
git tag
```

---

## 🏷️ Criar tag anotada
```bash
git tag -a 1.4.2 -m "Release 1.4.2"
git push origin 1.4.2
```


---

## 🗑️ Apagar tag local
```bash
git tag -d 1.4.2
```

---

## ☁️ Apagar tag remota
```bash
git push origin :refs/tags/1.4.2
```

---

# 📚 Resumo operacional

## Template A
- use para um serviço por repositório
- pode fazer copy/paste na maioria dos projetos
- ajuste só quando houver exceção técnica

## Template B
- use para monorepo com múltiplos serviços
- use quando existir bundle offline
- preserve a estrutura de empacotamento
- mantenha o padrão `edge/stable/version`

## Regra de bolso
- `main` = `edge`
- release estável = `stable` + versão
- pre-release = tag literal
- não usar `latest`

---

# ✅ Recomendação final

Mantenha estes dois modelos como padrão oficial:

- **Template A**: serviço único
- **Template B**: monorepo + bundle offline

Para projeto novo:
- se for simples, copy/paste do Template A
- se for solução completa com bundle, copy/paste do Template B

Não crie workflow novo do zero sem necessidade.
