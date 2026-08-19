# Análise Comparativa Aprofundada — Registries Públicos e Privados

## 1. Introdução

Esta análise compara cinco das principais plataformas de Container Registry — Docker
Hub, GitHub Container Registry (GHCR), Amazon ECR, Google Artifact Registry e Harbor —
sob os critérios exigidos pelo edital: **autenticação, versionamento, segurança,
integração com pipelines CI/CD e replicação de imagens**. O objetivo é orientar tanto
as conclusões teóricas do trabalho quanto o que deve ser observado no experimento
prático que compara um registry local a um registry público.

## 2. Critérios de Comparação

A comparação considera:

1. **Autenticação** — mecanismos suportados (credenciais, tokens, IAM/RBAC);
2. **Versionamento** — suporte a tags e digests, políticas de imutabilidade;
3. **Segurança** — escaneamento de vulnerabilidades, assinatura de imagens;
4. **Integração com CI/CD** — facilidade de uso em pipelines automatizados;
5. **Replicação** — capacidade de distribuir imagens entre regiões/instâncias.

## 3. Tabela Comparativa

| Solução | Autenticação | Versionamento | Segurança | Integração CI/CD | Replicação |
|---|---|---|---|---|---|
| **Docker Hub** | Usuário/senha, Personal Access Token | Tags e digests (padrão OCI) | Escaneamento limitado nos planos gratuitos | Ampla, via Docker CLI e Actions | Não nativa (depende de terceiros) |
| **GitHub Container Registry (GHCR)** | `GITHUB_TOKEN` automático em Actions, PAT | Tags e digests, visibilidade herdada do repositório | Via GitHub Advanced Security | Nativa e automática em GitHub Actions | Não nativa |
| **Amazon ECR** | IAM, tokens temporários (`aws ecr get-login-password`) | Tags e digests, políticas de ciclo de vida (lifecycle policies) | Nativo (Amazon Inspector) | Integração nativa com CodePipeline, ECS, EKS | Nativa (ECR replication entre regiões) |
| **Google Artifact Registry** | IAM, service accounts | Tags e digests, múltiplos formatos de artefato | Escaneamento automático nativo | Integração nativa com Cloud Build, GKE, Cloud Run | Repositórios regionais/multirregionais |
| **Harbor** | Local, LDAP/Active Directory, OIDC (SSO); RBAC granular por projeto | Tags e digests, controle rigoroso de imutabilidade | Nativo (Trivy) + assinatura (Notary/Cosign) | Configurável em qualquer pipeline via API/CLI | Nativa, entre múltiplas instâncias Harbor ou outros registries |

## 4. Discussão por Critério

### 4.1 Autenticação

Registries gerenciados por provedores de nuvem (ECR, Google AR) apoiam-se fortemente
em seus respectivos sistemas de IAM, o que reduz a superfície de gerenciamento manual
de credenciais, mas cria dependência do ecossistema do provedor. O GHCR se destaca
pela simplicidade em ambientes já centrados no GitHub, dispensando configuração
adicional de segredos em Actions. Harbor, por ser self-hosted, oferece a maior
flexibilidade de integração com sistemas corporativos existentes (LDAP/AD), a custo de
maior esforço operacional.

### 4.2 Versionamento

Todos os registries analisados seguem a especificação OCI Distribution e, portanto,
suportam tags e digests de forma equivalente em nível de protocolo. As diferenças
relevantes aparecem em funcionalidades adicionais: o Amazon ECR se destaca com
políticas de ciclo de vida para expurgo automático de imagens antigas, reduzindo custo
de armazenamento sem intervenção manual.

### 4.3 Segurança

Este é o critério de maior divergência entre as soluções. Amazon ECR, Google Artifact
Registry e Harbor oferecem escaneamento de vulnerabilidades nativo e integrado; Docker
Hub restringe essa funcionalidade nos planos gratuitos; e o GHCR depende da adoção do
GitHub Advanced Security. Harbor é a única solução, entre as analisadas, com suporte
nativo à assinatura de imagens (Notary/Cosign) já embutido na plataforma.

### 4.4 Integração com CI/CD

GHCR e as soluções de nuvem pública (ECR, Google AR) apresentam a integração mais
transparente quando o pipeline já reside no mesmo ecossistema (GitHub Actions, AWS
CodePipeline, Google Cloud Build, respectivamente). Docker Hub e Harbor são
agnósticos de plataforma, exigindo configuração explícita de credenciais em qualquer
pipeline, o que os torna mais portáveis entre diferentes provedores de CI/CD.

### 4.5 Replicação

Replicação nativa entre regiões ou instâncias é um diferencial claro de soluções
corporativas: Amazon ECR (entre regiões AWS), Google Artifact Registry (repositórios
regionais/multirregionais) e Harbor (entre instâncias Harbor ou outros registries).
Docker Hub e GHCR não oferecem esse recurso nativamente.

## 5. Orientação para o Experimento Prático

Com base nesta análise, recomenda-se que o experimento prático (registry local x
registry público) evidencie, no mínimo:

- O fluxo de autenticação e push/pull entre o registry local e o registry público
  escolhido (GHCR ou Docker Hub);
- A diferença prática entre referenciar uma imagem por tag e por digest;
- Uma comparação qualitativa do processo de configuração de credenciais entre o
  ambiente local e o serviço gerenciado.

## 6. Conclusão

Não existe uma solução única "melhor" entre os registries analisados: a escolha
depende do ecossistema de nuvem já adotado pela organização, dos requisitos de
segurança e compliance, e do modelo de infraestrutura desejado (gerenciado vs.
self-hosted). Times pequenos e projetos iniciais tendem a se beneficiar de registries
gerenciados (GHCR, ECR, Google AR), pela baixa fricção operacional, enquanto
organizações com requisitos de segurança e governança mais sofisticados encontram
maior valor em soluções como Harbor, que concentra RBAC granular, escaneamento,
assinatura e replicação em uma única plataforma self-hosted.