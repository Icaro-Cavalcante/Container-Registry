# Cenários de Aplicação e Boas Práticas — Container Registry

## 1. Introdução

A escolha e o uso de um Container Registry não são decisões puramente técnicas: elas
dependem do porte da equipe, do modelo de infraestrutura adotado (nuvem gerenciada vs.
*on-premise*) e do nível de maturidade em segurança exigido pela organização. Esta
seção consolida os cenários de aplicação mais comuns, com base no levantamento de
ferramentas (`docs/levantamento_ferramentas.md`) e na análise comparativa aprofundada
(`docs/analise_comparativa.md`), e propõe um checklist de boas práticas para o uso de
registries em pipelines de CI/CD — conectando a fundamentação teórica ao experimento
prático realizado com Docker Hub (`docs/guia_docker_hub.md`).

## 2. Cenários de Aplicação

### 2.1 Times pequenos e projetos iniciais

Equipes pequenas, projetos acadêmicos ou startups em fase inicial tendem a se
beneficiar de registries **gerenciados e de baixa fricção operacional**, como o Docker
Hub ou o GitHub Container Registry (GHCR):

- Não exigem infraestrutura própria para hospedar o registry;
- A curva de aprendizado é baixa, especialmente quando o time já usa GitHub como
  plataforma de versionamento (o GHCR dispensa configuração adicional de credenciais em
  GitHub Actions);
- O custo inicial é próximo de zero, já que os planos gratuitos atendem bem a projetos
  pequenos ou de código aberto.
- **Limitação a considerar**: escaneamento de vulnerabilidades mais completo costuma
  ficar restrito a planos pagos (caso do Docker Hub) ou a recursos adicionais (GitHub
  Advanced Security, no caso do GHCR).

Este é o cenário que motivou a escolha do **Docker Hub** como registry público no
experimento prático deste trabalho: baixa fricção de configuração para uma equipe
pequena, sem necessidade de infraestrutura própria.

### 2.2 Organizações com ecossistema de nuvem já definido

Empresas que já operam em um provedor de nuvem específico tendem a adotar o registry
nativo desse provedor — Amazon ECR (AWS), Google Artifact Registry (GCP) — pelos
seguintes motivos:

- Autenticação integrada ao IAM do provedor, eliminando a necessidade de gerenciar
  credenciais adicionais;
- Integração nativa com os serviços de orquestração e CI/CD do mesmo provedor (ECS/EKS
  e CodePipeline na AWS; GKE e Cloud Build no GCP);
- Escaneamento de vulnerabilidades e replicação entre regiões já embutidos no serviço,
  sem necessidade de ferramentas de terceiros.

### 2.3 Organizações com requisitos de segurança e compliance elevados

Organizações de médio/grande porte, ou que atuam em setores regulados, tendem a
priorizar soluções **self-hosted** com controle total sobre a infraestrutura, como o
Harbor:

- Permitem manter as imagens dentro do perímetro de rede da própria organização,
  atendendo a requisitos de compliance que exigem dados on-premise;
- Oferecem RBAC granular por projeto, integração com LDAP/Active Directory/OIDC já
  existentes na organização, escaneamento de vulnerabilidades nativo (Trivy) e suporte
  nativo à assinatura de imagens (Notary/Cosign);
- Em contrapartida, exigem maior esforço operacional (instalação, atualização,
  monitoramento da própria infraestrutura do registry).

### 2.4 On-premise vs. Cloud: síntese comparativa

| Critério | Registries gerenciados (Docker Hub, GHCR, ECR, GAR) | Registries self-hosted (Harbor, Nexus) |
|---|---|---|
| Esforço operacional | Baixo — serviço gerenciado pelo provedor | Alto — requer instalação, atualização e monitoramento próprios |
| Custo inicial | Baixo (planos gratuitos disponíveis) | Custo de infraestrutura própria |
| Controle sobre os dados | Dados hospedados pelo provedor | Dados dentro do perímetro da organização |
| Adequado para | Times pequenos, projetos que já usam o ecossistema do provedor | Organizações com requisitos de compliance e governança elevados |

## 3. Boas Práticas de Uso de Registries em Pipelines CI/CD

Com base na análise comparativa e na literatura consultada, consolidam-se as seguintes
boas práticas:

1. **Preferir digests a tags mutáveis em produção.** Tags como `latest` podem ser
   reapontadas a qualquer momento; referenciar a imagem pelo digest (`imagem@sha256:...`)
   garante que o ambiente de produção sempre execute exatamente a versão validada.
2. **Nunca versionar credenciais no código.** Tokens de autenticação (PAT, `GITHUB_TOKEN`,
   credenciais de IAM) devem ser armazenados como *secrets* do pipeline de CI/CD, nunca
   em texto plano no repositório.
3. **Escanear imagens antes do deploy.** Integrar uma etapa de escaneamento de
   vulnerabilidades (Docker Scout, Trivy, Clair, Amazon Inspector, conforme o registry
   utilizado) como *quality gate* do pipeline, bloqueando o avanço de imagens com
   vulnerabilidades críticas não corrigidas — prática cuja importância é reforçada pelo
   estudo de Shu, Gu e Enck (2017), que encontrou em média mais de 180 vulnerabilidades
   por imagem pública analisada no Docker Hub.
4. **Adotar uma convenção clara de versionamento.** Seguir Versionamento Semântico
   (`MAJOR.MINOR.PATCH`) para tags de release, evitando publicar apenas `latest`, o que
   dificulta rollback e rastreabilidade (ver `docs/versionamento.md`).
5. **Aplicar o princípio do menor privilégio na autenticação.** Utilizar tokens com
   escopo restrito (por repositório/projeto) em vez de credenciais de conta com acesso
   amplo, especialmente em pipelines automatizados.
6. **Automatizar a limpeza de imagens antigas.** Configurar políticas de ciclo de vida
   (*lifecycle policies*, disponíveis nativamente no Amazon ECR e configuráveis em
   Harbor) para expurgar imagens obsoletas e reduzir custo de armazenamento.
7. **Registrar procedência quando possível.** Em cenários que exigem maior garantia de
   integridade, considerar a assinatura de imagens (Sigstore/Cosign) e a geração de SBOM,
   conforme discutido em `docs/tendencias_desafios.md`.

## 4. Conclusão

Não existe um registry "ideal" de forma universal: a escolha correta depende do
equilíbrio entre esforço operacional, custo, requisitos de segurança/compliance e o
ecossistema de nuvem já adotado pela organização. Para o time deste trabalho — uma
equipe pequena, sem infraestrutura própria e já habituada ao Git/GitHub — o cenário
descrito na Seção 2.1 se aplica diretamente, o que justifica a escolha do Docker Hub
como registry público utilizado no experimento prático. Já os cenários das Seções 2.2
e 2.3 tornam-se mais relevantes à medida que a organização cresce e passa a operar em
um ecossistema de nuvem definido ou a lidar com requisitos regulatórios mais rígidos.

## 6. Referências

- SHU, R.; GU, X.; ENCK, W. **A Study of Security Vulnerabilities on Docker Hub**. In:
  Proceedings of the Seventh ACM Conference on Data and Application Security and
  Privacy (CODASPY '17), 2017. Disponível em:
  <https://dl.acm.org/doi/10.1145/3029806.3029832>.
  *(referência científica revisada por pares.)*
- OPEN CONTAINER INITIATIVE. **OCI Distribution Specification**. Disponível em:
  <https://github.com/opencontainers/distribution-spec>.
- DOCKER INC. **Docker Hub**. Disponível em: <https://hub.docker.com/>.
- SIGSTORE. **Cosign Quickstart**. Disponível em:
  <https://docs.sigstore.dev/quickstart/quickstart-cosign/>.

> Referências específicas sobre autenticação, segurança e versionamento estão detalhadas
> em `docs/seguranca_autenticacao.md` e `docs/versionamento.md`; a lista consolidada de
> referências bibliográficas de todo o trabalho está em `docs/referencias.md`.