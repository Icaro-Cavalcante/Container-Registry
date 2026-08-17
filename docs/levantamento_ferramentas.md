# Levantamento de Ferramentas e Soluções de Container Registry

## 1. Introdução

Existem hoje diversas soluções de Container Registry disponíveis no mercado, cada uma
com um posicionamento distinto: algumas são oferecidas como serviço gerenciado por
provedores de nuvem, outras são plataformas open source que podem ser hospedadas pela
própria organização (*self-hosted*), e outras ainda funcionam como registries
públicos genéricos, hospedando imagens tanto de projetos abertos quanto de repositórios
privados pagos.

Esta seção apresenta um levantamento das principais soluções utilizadas pela indústria,
servindo de base para a análise comparativa aprofundada que será conduzida
posteriormente (ver `docs/analise_comparativa.md`, de responsabilidade de ELI).

## 2. Principais Soluções

### 2.1 Docker Hub

O **Docker Hub** é o registry público mais conhecido e o padrão *default* utilizado
pelo Docker CLI quando nenhum outro registry é especificado. Mantido pela Docker, Inc.,
hospeda milhões de imagens, incluindo as "Official Images" (imagens oficiais mantidas
e verificadas para tecnologias amplamente usadas, como `nginx`, `postgres`, `python`).

- **Tipo**: público e privado (planos pagos para repositórios privados e maior limite
  de *pulls*);
- **Autenticação**: usuário/senha ou token de acesso pessoal (*Personal Access Token*);
- **Diferenciais**: maior base de imagens públicas do ecossistema; imagens oficiais
  com garantias de segurança e manutenção; integração nativa com o Docker CLI.

### 2.2 GitHub Container Registry (GHCR)

O **GHCR** é o registry de containers integrado ao GitHub, permitindo publicar imagens
associadas diretamente a um repositório ou a uma organização do GitHub.

- **Tipo**: público e privado;
- **Autenticação**: tokens do próprio GitHub (Personal Access Token ou
  `GITHUB_TOKEN` automático em GitHub Actions);
- **Diferenciais**: integração nativa com GitHub Actions, facilitando pipelines de
  CI/CD que já usam o GitHub como plataforma de versionamento de código; controle de
  visibilidade (público/privado) e permissões herdados da própria organização/
  repositório do GitHub; sem necessidade de configurar credenciais adicionais quando
  usado dentro de workflows do próprio GitHub.

### 2.3 Amazon Elastic Container Registry (Amazon ECR)

O **ECR** é o serviço de registry gerenciado da AWS, projetado para se integrar de
forma transparente com os demais serviços do ecossistema AWS (ECS, EKS, Lambda por
container image, CodePipeline).

- **Tipo**: privado (por padrão), com opção de registries públicos via ECR Public;
- **Autenticação**: integrada ao IAM (Identity and Access Management) da AWS, via
  tokens temporários obtidos pelo AWS CLI (`aws ecr get-login-password`);
- **Diferenciais**: escaneamento de vulnerabilidades integrado (via Amazon Inspector);
  replicação entre regiões; políticas de ciclo de vida (*lifecycle policies*) para
  expurgo automático de imagens antigas; cobrança por uso (armazenamento e
  transferência).

### 2.4 Google Artifact Registry (GAR)

O **Google Artifact Registry** é o serviço da Google Cloud sucessor do antigo
*Google Container Registry (GCR)*, ampliado para armazenar não apenas imagens de
container, mas também outros tipos de artefatos (pacotes npm, Maven, Python, etc.).

- **Tipo**: privado, com controle refinado de visibilidade;
- **Autenticação**: integrada ao IAM do Google Cloud, usando contas de serviço
  (*service accounts*) e o utilitário `gcloud auth configure-docker`;
- **Diferenciais**: repositórios regionais ou multirregionais; escaneamento automático
  de vulnerabilidades; integração nativa com Cloud Build, GKE (Google Kubernetes
  Engine) e Cloud Run.

### 2.5 Harbor

O **Harbor** é uma plataforma de registry **open source**, originalmente desenvolvida
pela VMware e hoje mantida como projeto graduado da CNCF (Cloud Native Computing
Foundation). É voltada para organizações que desejam hospedar seu próprio registry
privado (*self-hosted*), com foco em segurança e governança.

- **Tipo**: privado, self-hosted;
- **Autenticação**: suporte a usuários locais, LDAP/Active Directory e OIDC
  (Single Sign-On);
- **Diferenciais**: controle de acesso baseado em papéis (RBAC) por projeto;
  escaneamento de vulnerabilidades integrado (Trivy); replicação de imagens entre
  múltiplas instâncias de Harbor ou outros registries; assinatura de imagens (Notary/
  Cosign); interface web completa para gestão de projetos e políticas.

### 2.6 Sonatype Nexus Repository

O **Nexus Repository** é um gerenciador de artefatos genérico e self-hosted, que
suporta não apenas imagens Docker/OCI, mas também outros formatos de pacotes (Maven,
npm, PyPI, NuGet, etc.) em um único servidor.

- **Tipo**: privado, self-hosted (com versão comunitária gratuita e versão Pro paga);
- **Autenticação**: usuários locais, LDAP/Active Directory, controle de acesso por
  realm e por repositório;
- **Diferenciais**: consolidação de múltiplos tipos de artefato em uma única
  plataforma, o que é útil para organizações que já usam o Nexus para gerenciar
  dependências de build (Maven/npm) e desejam adicionar imagens de container ao mesmo
  ambiente; suporte a repositórios "proxy" (cache de registries externos) e
  "group" (agregação de múltiplos repositórios).

### 2.7 Red Hat Quay

O **Quay** (também disponível como *Quay.io*, versão hospedada, e como *Red Hat Quay*,
versão self-hosted/enterprise) é um registry com forte ênfase em segurança.

- **Tipo**: público (Quay.io) e privado (Red Hat Quay, self-hosted);
- **Autenticação**: usuários locais, OAuth, integração com Red Hat Single Sign-On;
- **Diferenciais**: escaneamento de vulnerabilidades integrado (Clair); assinatura de
  imagens; geo-replicação; integração nativa com o ecossistema Red Hat/OpenShift.

## 3. Visão Geral Comparativa (Resumo)

| Solução | Tipo | Hospedagem | Autenticação principal | Escaneamento de vulnerabilidades |
|---|---|---|---|---|
| Docker Hub | Público/Privado | SaaS | Usuário/senha, PAT | Limitado nos planos gratuitos |
| GHCR | Público/Privado | SaaS (GitHub) | Tokens do GitHub | Via GitHub Advanced Security |
| Amazon ECR | Privado (+ Public) | SaaS (AWS) | IAM | Nativo (Amazon Inspector) |
| Google Artifact Registry | Privado | SaaS (GCP) | IAM/Service Account | Nativo |
| Harbor | Privado | Self-hosted | Local/LDAP/OIDC | Nativo (Trivy) |
| Nexus Repository | Privado | Self-hosted | Local/LDAP | Via plugins/integrações |
| Quay | Público/Privado | SaaS ou Self-hosted | Local/OAuth/SSO | Nativo (Clair) |

> Esta tabela é um resumo inicial de caráter introdutório. A análise comparativa
> aprofundada — cobrindo autenticação, versionamento, segurança, integração com
> pipelines CI/CD e replicação de imagens, conforme exigido pelo edital — será
> detalhada em `docs/analise_comparativa.md` (ELI-03), com base neste levantamento.

## 4. Considerações Finais

O levantamento evidencia que a escolha de um Container Registry não é apenas uma
decisão técnica, mas também estratégica: envolve considerar o ecossistema de nuvem já
adotado pela organização (AWS, GCP, GitHub), a necessidade de hospedagem própria por
requisitos de compliance (Harbor, Nexus, Quay self-hosted) e o nível de maturidade em
segurança exigido (escaneamento de vulnerabilidades, assinatura de imagens, RBAC). Esse
panorama servirá de insumo direto para os cenários de aplicação e boas práticas a serem
consolidados posteriormente em `docs/cenarios_boas_praticas.md` (ASH-03).