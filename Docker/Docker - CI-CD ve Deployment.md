---
tarih: 2025-01-01
konu: Docker ile CI/CD, GitHub Actions, GitLab CI, Deployment Stratejileri
etiket: [docker, ci-cd, github-actions, gitlab, deployment, rolling-update]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

Docker, modern yazılım geliştirme süreçlerinin ayrılmaz bir parçası olan CI/CD (Sürekli Entegrasyon ve Sürekli Dağıtım) boru hatlarının temel taşıdır. Konteyner yapısı sayesinde, bir uygulamanın build, test ve deployment aşamaları tamamen izole ve tekrarlanabilir bir ortamda gerçekleştirilebilir. GitHub Actions veya GitLab CI gibi araçlarla entegre edilen Docker, kod değişikliğinden itibaren otomatik imaj oluşturma, güvenlik taraması ve sıfır kesinti (zero-downtime) ile yayına alma süreçlerini mümkün kılar. Bu otomasyon, geliştirme ekiplerinin hataları erkenden tespit etmesini sağlar ve yazılımın üretim ortamına güvenle, hızla ve standart bir şekilde taşınmasını garanti eder.

---

## 🧠 Detay

```mermaid
graph LR
    A["Kod Değişikliği (Push)"] --> B["CI Sunucusu (GitHub/GitLab)"]
    B --> C["Otomatik Testler"]
    C -->|"Başarılı"| D["Docker Build & Tag"]
    D --> E["İmaj Güvenlik Taraması"]
    E -->|"Güvenli"| F["Docker Registry'ye Push"]
    F --> G["Deployment (CD)"]
    G --> H["Rolling Update / Blue-Green"]
    subgraph "Sürekli Entegrasyon (CI)"
    B
    C
    D
    E
    end
    subgraph "Sürekli Dağıtım (CD)"
    F
    G
    H
    end
```

### GitHub Actions ile Docker CI/CD

```yaml
# .github/workflows/docker-cicd.yml
name: Docker CI/CD

on:
  push:
    branches: [main, develop]
    tags: ['v*.*.*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ── Test ──────────────────────────────────
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: Python kur
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Bağımlılıkları yükle
        run: pip install -r requirements-dev.txt

      - name: Testleri çalıştır
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
        run: pytest --cov=app --cov-report=xml

      - name: Coverage yükle
        uses: codecov/codecov-action@v3

  # ── Build & Push ───────────────────────────
  build-push:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Docker Buildx kur
        uses: docker/setup-buildx-action@v3

      - name: Registry giriş
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Image metadata
        uses: docker/metadata-action@v5
        id: meta
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix=sha-

      - name: Build & Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_DATE=${{ github.event.repository.updated_at }}
            GIT_HASH=${{ github.sha }}

  # ── Güvenlik Tarama ───────────────────────
  security-scan:
    needs: build-push
    runs-on: ubuntu-latest
    steps:
      - name: Trivy ile tara
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main
          format: sarif
          output: trivy-results.sarif
          severity: HIGH,CRITICAL

      - name: SARIF yükle
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: trivy-results.sarif

  # ── Deploy (Production) ───────────────────
  deploy-prod:
    needs: [build-push, security-scan]
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production

    steps:
      - name: SSH ile deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/myapp
            docker compose pull
            docker compose up -d --no-deps web
            docker system prune -f
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
variables:
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG

stages:
  - test
  - build
  - scan
  - deploy

test:
  stage: test
  image: python:3.11-slim
  services:
    - postgres:15-alpine
  variables:
    POSTGRES_PASSWORD: testpass
    POSTGRES_DB: testdb
    DATABASE_URL: postgresql://postgres:testpass@postgres/testdb
  script:
    - pip install -r requirements-dev.txt
    - pytest --cov=app

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker buildx build
        --cache-from $IMAGE_TAG
        --tag $IMAGE_TAG
        --push .

trivy-scan:
  stage: scan
  image: aquasec/trivy
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_TAG

deploy-staging:
  stage: deploy
  environment: staging
  only:
    - develop
  script:
    - ssh deploy@staging.example.com "
        docker compose pull &&
        docker compose up -d --no-deps web"

deploy-prod:
  stage: deploy
  environment: production
  only:
    - tags
  when: manual
  script:
    - ssh deploy@prod.example.com "
        docker compose pull &&
        docker compose up -d --no-deps web"
```

### Deployment Stratejileri

#### Rolling Update (Swarm)

```bash
docker service update \
  --image myapp:2.0 \
  --update-parallelism 1 \
  --update-delay 30s \
  --update-order start-first \
  --update-failure-action rollback \
  myapp_web
```

#### Blue-Green Deployment

```bash
# Mevcut: blue (production)
# Yeni: green (test)

# Green'i başlat
docker compose -f docker-compose.green.yml up -d

# Sağlık kontrolü
curl http://green.internal:8000/health

# Nginx'i green'e yönlendir
sed -i 's/blue:8000/green:8000/' /etc/nginx/nginx.conf
nginx -s reload

# Blue'yu kapat
docker compose -f docker-compose.blue.yml down

# Sorun olursa geri al
sed -i 's/green:8000/blue:8000/' /etc/nginx/nginx.conf
nginx -s reload
```

#### Canary Deployment (Nginx ile)

```nginx
# nginx.conf — %10 trafiği yeni versiyona
upstream backend {
    server old-app:8000 weight=9;  # %90
    server new-app:8000 weight=1;  # %10
}
```

### Otomatik Rollback Script

```bash
#!/bin/bash
# deploy.sh

IMAGE="myregistry/myapp"
NEW_TAG=$1
OLD_TAG=$(docker inspect myapp_web --format='{{.Config.Image}}' | cut -d: -f2)

echo "Deploying: $NEW_TAG (previous: $OLD_TAG)"

# Yeni image'ı indir
docker pull $IMAGE:$NEW_TAG

# Güncelle
docker service update --image $IMAGE:$NEW_TAG myapp_web

# 60 saniye bekle ve sağlığı kontrol et
sleep 60
HEALTH=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/health)

if [ "$HEALTH" != "200" ]; then
  echo "Health check failed! Rolling back to $OLD_TAG"
  docker service update --image $IMAGE:$OLD_TAG myapp_web
  exit 1
fi

echo "Deployment successful!"
```

### Docker Build Optimizasyonu (CI için)

```yaml
# GitHub Actions — maksimum cache kullanımı
- name: Build & Push
  uses: docker/build-push-action@v5
  with:
    context: .
    cache-from: |
      type=gha
      type=registry,ref=${{ env.IMAGE_NAME }}:buildcache
    cache-to: |
      type=gha,mode=max
      type=registry,ref=${{ env.IMAGE_NAME }}:buildcache,mode=max
    platforms: linux/amd64,linux/arm64  # Çok platform
```

### Ortam Değişkenleri Yönetimi

```bash
# .env.example (repoya ekle)
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
SECRET_KEY=change-me-in-production
DEBUG=False
ALLOWED_HOSTS=localhost

# .env (gitignore — gerçek değerler)
DATABASE_URL=postgresql://produser:secretpass@prod-db:5432/proddb
SECRET_KEY=a-very-long-random-secret-key-here
DEBUG=False
ALLOWED_HOSTS=myapp.com,www.myapp.com
```

```yaml
# compose.yml — env_file kullanımı
services:
  web:
    env_file:
      - .env
    # Veya CI'da ortam değişkenlerini doğrudan geç:
    environment:
      - DATABASE_URL
      - SECRET_KEY
```

---

## 💡 Bağlantılar
- [[Docker - Registry ve Image Yönetimi]]
- [[Docker - Swarm ve Ölçeklendirme]]
- [[Docker - Güvenlik Best Practices]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- GitHub Actions Docker Documentation
- GitLab CI/CD Docker Integration
- docs.docker.com/build/ci/
