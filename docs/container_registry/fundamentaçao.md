# Fundamentação Teórica — Registro de Imagens (Container Registry)

## 1. Introdução

A popularização de containers como unidade padrão de empacotamento e execução de
aplicações trouxe a necessidade de um componente capaz de armazenar, versionar e
distribuir as imagens geradas durante o ciclo de desenvolvimento. Esse componente é o
**Container Registry** (Registro de Imagens): um serviço responsável por receber,
armazenar e disponibilizar imagens de container para os ambientes que precisam
executá-las — desenvolvimento, testes, homologação e produção.

Sem um registry, cada imagem construída localmente teria que ser transportada
manualmente entre máquinas, o que inviabiliza práticas essenciais de DevOps como
Integração Contínua e Entrega Contínua (CI/CD), reprodutibilidade de ambientes e
rastreabilidade de versões.

## 2. O que é um Container Registry

Um Container Registry é, em termos simples, um **servidor de artefatos especializado em
imagens de container**. Ele expõe uma API HTTP (o padrão mais adotado é a
*Docker/OCI Distribution Specification*) através da qual clientes — como o Docker CLI,
o `nerdctl`, o `podman` ou pipelines de CI/CD — podem:

- **Enviar (push)** uma imagem recém-construída para o registry;
- **Baixar (pull)** uma imagem já armazenada para executá-la localmente ou em um
  cluster;
- **Listar e consultar** as tags e versões disponíveis de um repositório de imagens;
- **Autenticar e autorizar** o acesso, garantindo que apenas usuários e serviços
  legítimos possam publicar ou consumir determinadas imagens.

Um registry pode ser **público** (acessível por qualquer pessoa, geralmente para
imagens open source) ou **privado** (restrito a uma organização ou equipe, comum para
aplicações proprietárias).

## 3. Papel do Registry no Ecossistema de Containers

O Container Registry ocupa uma posição central no fluxo de trabalho de aplicações
conteinerizadas, funcionando como ponte entre a fase de **construção (build)** e a fase
de **execução (run)**:

```
Código-fonte -> Build (Dockerfile) -> Imagem local -> PUSH -> Registry -> PULL -> Ambiente de execução
                                                                              (Docker, Kubernetes, etc.)
```

Esse papel o torna um elemento crítico em qualquer pipeline de CI/CD: é a partir do
registry que ambientes de teste, homologação e produção obtêm exatamente a mesma
imagem gerada e validada anteriormente, o que garante consistência entre ambientes —
um dos princípios centrais da Gerência de Configuração.

Além disso, o registry viabiliza práticas como:

- **Orquestração**: plataformas como o Kubernetes fazem *pull* de imagens diretamente
  do registry configurado para provisionar os Pods de uma aplicação;
- **Versionamento de artefatos**: cada imagem publicada pode ser associada a uma tag ou
  a um número de versão, permitindo rollback rápido para uma versão anterior estável;
- **Auditoria e segurança**: registries modernos oferecem escaneamento de
  vulnerabilidades, controle de acesso granular e trilhas de auditoria sobre quem
  publicou ou consumiu determinada imagem. Esse cuidado não é apenas teórico: um estudo
  acadêmico que analisou mais de 350 mil imagens do Docker Hub encontrou, em média,
  mais de 180 vulnerabilidades por imagem, muitas delas propagadas de imagens-pai para
  imagens-filha sem atualização (SHU; GU; ENCK, 2017), o que reforça a importância do
  papel de auditoria e escaneamento desempenhado pelo registry.

## 4. Conceitos Fundamentais

### 4.1 Imagem (Image)

Uma imagem de container é um pacote imutável e somente leitura que contém tudo o que é
necessário para executar uma aplicação: código, runtime, bibliotecas, variáveis de
ambiente e arquivos de configuração. É a partir de uma imagem que um ou mais
**containers** (instâncias em execução) são criados.

### 4.2 Camadas (Layers)

Internamente, uma imagem não é um bloco único de dados, mas sim uma **pilha de
camadas** (layers), onde cada camada representa uma instrução do Dockerfile (por
exemplo, `RUN`, `COPY`, `ADD`), conforme descrito na
[documentação oficial do Docker sobre storage drivers](https://docs.docker.com/storage/storagedriver/).
Esse modelo em camadas traz vantagens importantes para o registry:

- **Reuso e cache**: camadas idênticas entre imagens diferentes são armazenadas apenas
  uma vez, economizando espaço em disco e banda de rede;
- **Transferência incremental**: ao fazer *pull* de uma nova versão de uma imagem, o
  cliente só precisa baixar as camadas que mudaram, não a imagem inteira;
- **Imutabilidade**: uma vez criada, uma camada nunca é alterada; qualquer mudança gera
  uma nova camada, o que contribui para a rastreabilidade da imagem.

### 4.3 Manifest

O **manifest** é um documento (em formato JSON) que descreve a estrutura de uma
imagem: quais camadas a compõem, em que ordem devem ser aplicadas, qual a arquitetura
de CPU/SO-alvo (por exemplo, `linux/amd64` ou `linux/arm64`) e metadados adicionais.
Quando um cliente solicita o *pull* de uma imagem, o registry primeiro entrega o
manifest, e só então o cliente baixa as camadas referenciadas nele que ainda não
possui localmente.

Quando uma mesma tag precisa suportar múltiplas arquiteturas (multi-arch), o registry
utiliza um **manifest list** (ou *image index*), que aponta para diferentes manifests
específicos de cada plataforma.

### 4.4 Tags e Digests

- **Tag**: um rótulo legível por humanos associado a uma versão da imagem (por exemplo,
  `minha-app:1.2.0` ou `minha-app:latest`). Tags são mutáveis — podem ser reapontadas
  para um novo conteúdo a qualquer momento, o que exige cuidado em ambientes de
  produção.
- **Digest**: um hash criptográfico (SHA-256) calculado sobre o conteúdo do manifest.
  Diferente da tag, o digest é **imutável** e identifica de forma única e verificável
  uma versão exata da imagem, sendo a referência recomendada quando se busca
  reprodutibilidade e segurança (por exemplo, `minha-app@sha256:abcd1234...`).

### 4.5 Padrão OCI (Open Container Initiative)

A **OCI** é uma organização, mantida sob a Linux Foundation, criada para padronizar os
formatos de imagem e runtime de containers, evitando o *lock-in* em uma implementação
específica (como o formato proprietário original do Docker). Ela define principalmente
três especificações relevantes para este trabalho:

- **[OCI Image Format Specification](https://github.com/opencontainers/image-spec)**:
  define a estrutura de uma imagem (manifest, camadas, configuração);
- **[OCI Distribution Specification](https://github.com/opencontainers/distribution-spec)**:
  define a API HTTP usada por clientes e registries para push/pull de imagens — é essa
  especificação que garante que ferramentas como Docker, Podman e diferentes registries
  (Docker Hub, GHCR, Harbor etc.) sejam interoperáveis entre si;
- **OCI Runtime Specification**: define como um container deve ser executado a partir
  de uma imagem descompactada.

Graças à padronização da OCI, uma imagem publicada em um registry pode ser consumida
por qualquer cliente compatível, independentemente de qual ferramenta a construiu ou
qual registry a armazena — o que é essencial para o ecossistema de Gerência de
Configuração e DevOps abordado neste trabalho.

## 5. Referências

- OPEN CONTAINER INITIATIVE. **OCI Distribution Specification**. Disponível em:
  <https://github.com/opencontainers/distribution-spec>.
- OPEN CONTAINER INITIATIVE. **OCI Image Format Specification**. Disponível em:
  <https://github.com/opencontainers/image-spec>.
- DOCKER INC. **About storage drivers** (camadas e arquitetura de imagens). Docker
  Documentation. Disponível em: <https://docs.docker.com/storage/storagedriver/>.
- SHU, R.; GU, X.; ENCK, W. **A Study of Security Vulnerabilities on Docker Hub**. In:
  Proceedings of the Seventh ACM Conference on Data and Application Security and
  Privacy (CODASPY '17), 2017. Disponível em:
  <https://dl.acm.org/doi/10.1145/3029806.3029832>.
  *(referência científica revisada por pares, utilizada para embasar a discussão sobre
  o papel do registry na segurança da cadeia de distribuição de imagens.)*'
