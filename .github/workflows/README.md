# CI/CD Workflows

Este diretório contém o workflow do GitHub Actions do `evaluation-service` (`ci.yml`), disparado em `push` e `pull_request` para a branch `main`. Os jobs rodam em sequência via `needs`, então uma falha em um estágio impede os estágios seguintes de rodar.

## `build-test` — Build & Unit Test

- Configura o Go a partir do `go.mod`.
- Baixa as dependências (`go mod download`).
- Compila o projeto (`go build ./...`).
- Executa os testes unitários (`go test ./...`).

## `lint` — Lint & Static Analysis

- Executa `golangci-lint` para checagem de estilo e qualidade do código Go.

## `security-scan` — Security Scan (SAST & SCA)

Depende de `build-test` e `lint`.

- Roda `gosec` (SAST) contra o código-fonte com severidade e confiança mínimas `high`.
- Roda `trivy` (SCA) em modo filesystem para vulnerabilidades `CRITICAL` nas dependências.

## `gitleaks` — Secret Scanning

- Escaneia o histórico do repositório em busca de segredos vazados (chaves, tokens, credenciais).

## `docker-build-push` — Docker Build & Push

Depende de `security-scan` e `gitleaks` — só roda se ambos passarem.

- Faz lint do `Dockerfile` com `hadolint`.
- Gera uma tag de imagem no formato `v1.0.0-<sha curto>`.
- Constrói a imagem Docker e roda `trivy` em modo image scan (`CRITICAL,HIGH`).
- Em eventos de `push` para `main`: autentica na AWS, faz login no Amazon ECR e publica a imagem no repositório `evaluation-service`.
- Em seguida, faz checkout do repositório [`fiap-challenge-3-gitops`](https://github.com/millenevprado/fiap-challenge-3-gitops), atualiza a linha `image:` do `evaluation-service/deployment.yaml` com a nova tag e faz commit/push automático (entrega contínua).

### Secrets necessários

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `GITOPS_PAT` — Personal Access Token com permissão de escrita (`Contents: Read and write`) no repositório `fiap-challenge-3-gitops`.
