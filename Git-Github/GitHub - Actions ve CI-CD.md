---
tarih: 2025-01-01
konu: GitHub Actions, CI-CD, Workflow, Jobs, Secrets, Matrix Build
etiket: [github, actions, ci-cd, workflow, pipeline, secrets, matrix]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

GitHub Actions, kod değişikliğinden production'a kadar tüm süreci otomatikleştiren CI/CD platformudur. Workflow, Job, Step ve Action kavramları üzerine kuruludur.

---

## 🧠 Detay

### Temel Kavramlar

```
Event (tetikleyici)
   │
   ▼
Workflow (.github/workflows/*.yml)
   │
   ├─ Job 1 (ubuntu-latest)
   │    ├─ Step 1: Checkout
   │    ├─ Step 2: Setup Python
   │    └─ Step 3: Run tests
   │
   └─ Job 2 (ubuntu-latest) ← Job 1'e bağımlı
        ├─ Step 1: Build Docker
        └─ Step 2: Push to registry
```

### Tetikleyiciler (Triggers)

```yaml
on:
  # Push olayı
  push:
    branches: [main, develop]
    paths:
      - "src/**"
      - "!docs/**"          # Docs değişiminde tetiklenme

  # PR olayı
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # Manuel tetikleme
  workflow_dispatch:
    inputs:
      environment:
        description: "Deploy ortamı"
        required: true
        default: staging
        type: choice
        options: [staging, production]
      debug:
        type: boolean
        default: false

  # Zamanlı (CRON)
  schedule:
    - cron: "0 2 * * 1"   # Her Pazartesi 02:00 UTC

  # Başka workflow'u tetikle
  workflow_call:
    inputs:
      version:
        type: string
        required: true

  # Release
  release:
    types: [published]

  # Issue olayları
  issues:
    types: [opened, labeled]
```

### Python CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: "3.11"
  POETRY_VERSION: "1.7.0"

jobs:
  test:
    name: Test (${{ matrix.python-version }})
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
      fail-fast: false        # Bir versiyon başarısız olsa diğerleri devam

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - name: Kodu indir
        uses: actions/checkout@v4

      - name: Python ${{ matrix.python-version }} kur
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - name: Bağımlılıkları yükle
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt

      - name: Linting
        run: |
          flake8 src/ tests/
          black --check src/ tests/
          isort --check-only src/ tests/
          mypy src/

      - name: Testleri çalıştır
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
          SECRET_KEY: test-secret-key-for-ci
        run: |
          pytest tests/ \
            --cov=src \
            --cov-report=xml \
            --cov-report=html \
            -v \
            --tb=short

      - name: Coverage raporu yükle
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: false

  security:
    name: Güvenlik Taraması
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Bandit (Python güvenlik)
        run: |
          pip install bandit
          bandit -r src/ -f json -o bandit-report.json || true
      - name: Safety (bağımlılık güvenlik)
        run: |
          pip install safety
          safety check

  build:
    name: Docker Build
    runs-on: ubuntu-latest
    needs: [test]              # test job'u başarılı olmalı
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Docker Buildx kur
        uses: docker/setup-buildx-action@v3

      - name: Docker Hub giriş
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Metadata hazırla
        uses: docker/metadata-action@v5
        id: meta
        with:
          images: ${{ secrets.DOCKER_USERNAME }}/myapp
          tags: |
            type=sha,prefix=sha-
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Build ve Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Deploy Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        type: environment
        required: true

jobs:
  deploy:
    name: Deploy to ${{ inputs.environment }}
    runs-on: ubuntu-latest
    environment:
      name: ${{ inputs.environment }}
      url: ${{ steps.deploy.outputs.url }}

    steps:
      - uses: actions/checkout@v4

      - name: AWS Kimlik bilgilerini ayarla
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-west-1

      - name: ECR'a giriş yap
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Image build ve push
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/myapp:$IMAGE_TAG .
          docker push $ECR_REGISTRY/myapp:$IMAGE_TAG

      - name: ECS servisi güncelle
        id: deploy
        run: |
          aws ecs update-service \
            --cluster myapp-${{ inputs.environment }} \
            --service myapp \
            --force-new-deployment
          echo "url=https://${{ inputs.environment }}.myapp.com" >> $GITHUB_OUTPUT

      - name: Deployment bildir
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: "deployments"
          slack-message: "✅ ${{ inputs.environment }} ortamına deploy edildi!"
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test

on:
  workflow_call:
    inputs:
      python-version:
        type: string
        default: "3.11"
    secrets:
      codecov-token:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
      - run: pytest

---
# Ana workflow içinde kullan:
jobs:
  call-test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      python-version: "3.12"
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}
```

### Composite Actions

```yaml
# .github/actions/setup-python-poetry/action.yml
name: "Setup Python + Poetry"
description: "Python ve Poetry kurulumu"

inputs:
  python-version:
    description: "Python versiyonu"
    required: false
    default: "3.11"

outputs:
  venv-path:
    description: "Virtual environment yolu"
    value: ${{ steps.setup.outputs.venv }}

runs:
  using: "composite"
  steps:
    - uses: actions/setup-python@v5
      with:
        python-version: ${{ inputs.python-version }}

    - name: Poetry kur
      shell: bash
      run: |
        pip install poetry
        poetry config virtualenvs.in-project true

    - name: Bağımlılıkları yükle
      id: setup
      shell: bash
      run: |
        poetry install
        echo "venv=$(poetry env info --path)" >> $GITHUB_OUTPUT

# Kullanım:
# - uses: ./.github/actions/setup-python-poetry
#   with:
#     python-version: "3.11"
```

### Secrets ve Environment Variables

```yaml
# Secret kullanımı
steps:
  - name: API'ye bağlan
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
    run: python deploy.py

# Environment secrets (ortama özel)
# GitHub → Settings → Environments → staging/production → Secrets

# Maskeleme (log'larda gizle)
- name: Secret maskeleme
  run: |
    TOKEN=$(generate_token)
    echo "::add-mask::$TOKEN"   # Bu değer log'larda *** olarak görünür
    echo "token=$TOKEN" >> $GITHUB_OUTPUT

# Output variables
- name: Versiyon belirle
  id: version
  run: |
    VERSION=$(cat VERSION)
    echo "version=$VERSION" >> $GITHUB_OUTPUT

- name: Versiyonu kullan
  run: echo "Versiyon: ${{ steps.version.outputs.version }}"
```

### Matrix Strategy

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python: ["3.10", "3.11", "3.12"]
        include:
          - os: ubuntu-latest
            python: "3.12"
            experimental: true       # Ekstra değişken
        exclude:
          - os: windows-latest
            python: "3.10"           # Bu kombinasyonu atla

      fail-fast: false               # Biri başarısız olsa diğerleri devam
      max-parallel: 4                # Eş zamanlı max 4

    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python }}
```

### Önbellekleme (Cache)

```yaml
# pip cache
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-

# node_modules cache
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Docker layer cache (BuildKit)
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Hızlı Başvuru

```yaml
# Koşullar
if: github.event_name == 'push'
if: github.ref == 'refs/heads/main'
if: contains(github.event.pull_request.labels.*.name, 'deploy')
if: failure()                        # Önceki adım başarısız oldu
if: success() && github.ref == 'refs/heads/main'
if: always()                         # Her durumda çalıştır

# Kontekst değişkenleri
${{ github.sha }}                    # Commit hash
${{ github.ref }}                    # refs/heads/main
${{ github.actor }}                  # Tetikleyen kullanıcı
${{ github.repository }}             # user/repo
${{ runner.os }}                     # Linux/macOS/Windows
${{ env.MY_VAR }}                    # Ortam değişkeni

# Job bağımlılığı
needs: [test, lint]
needs: test
```

---

## 💡 Bağlantılar
- [[GitHub - Repository Yönetimi]]
- [[GitHub - Pull Request ve Code Review]]
- [[Docker - Registry ve Image Yönetimi]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- docs.github.com/en/actions
- github.com/marketplace?type=actions
