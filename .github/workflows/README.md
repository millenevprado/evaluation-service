# CI/CD Workflows

Este diretório contém os workflows do GitHub Actions do `evaluation-service`. Todos são disparados em `push` e `pull_request` para a branch `main`.

## `build-test.yml` — Build & Unit Test

- Configura o Go a partir do `go.mod`.
- Baixa as dependências (`go mod download`).
- Compila o projeto (`go build ./...`).
- Executa os testes unitários (`go test ./...`).

## `lint.yml` — Lint & Static Analysis

- Executa `golangci-lint` para checagem de estilo e qualidade do código Go.

## `security-scan.yml` — Security Scan (SAST & SCA)

Dois jobs independentes:

- **security-scan**: roda `gosec` (SAST) contra o código-fonte com severidade e confiança mínimas `high`, e `trivy` (SCA) em modo filesystem para vulnerabilidades `CRITICAL` nas dependências.
- **gitleaks**: escaneia o histórico do repositório em busca de segredos vazados (chaves, tokens, credenciais).

## `docker-build-push.yml` — Docker Build & Push

- Faz lint do `Dockerfile` com `hadolint`.
- Gera uma tag de imagem no formato `v1.0.0-<sha curto>`.
- Constrói a imagem Docker e roda `trivy` em modo image scan (`CRITICAL,HIGH`).
- Em eventos de `push` para `main`: autentica na AWS, faz login no Amazon ECR e publica a imagem no repositório `evaluation-service`.

### Secrets necessários

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
