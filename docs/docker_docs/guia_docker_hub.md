# Guia Prático: Docker Hub e Docker Desktop para Registro de Imagens

*Instruções conceituais e práticas para armazenamento, versionamento e distribuição de containers utilizando o Docker Hub e auxiliado pela aplicação Docker Desktop.*

---

## 1. Introdução e Contextualização

No ciclo de vida do desenvolvimento de software baseado em containers, o **Container Registry** (Registro de Imagens) é a peça central que conecta a etapa de construção (*build*) à etapa de execução (*run* ou *deploy*). Sem um registry centralizado, compartilhar artefatos conteinerizados entre desenvolvedores, servidores de teste e ambientes de produção exigiria transferências manuais de arquivos, comprometendo a reprodutibilidade e a automação essenciais da **Gerência de Configuração** e de práticas **DevOps**.

O **Docker Hub** é o serviço de Container Registry em nuvem (*Software as a Service - SaaS*) padrão mantido pela Docker, Inc. Trata-se da plataforma mais popular e amplamente utilizada no ecossistema de containers, servindo tanto como repositório central de imagens comunitárias e oficiais (como `nginx`, `gcc`, `python`, `debian`), quanto como registry privado para desenvolvedores e organizações.

### Papel do Docker Hub no Projeto

Neste projeto de estudo sobre *Container Registries*, o Docker Hub representa a categoria de **Registry Público/Gerenciado (SaaS)**, servindo como modelo de referência para entender:
- Fluxo de publicação e distribuição de imagens (`push` e `pull`);
- Versionamento semântico por meio de tags e imutabilidade por digests (SHA-256);
- Mecanismos de autenticação baseados em *Personal Access Tokens* (PAT);
- Gerenciamento de artefatos tanto por interface gráfica (**Docker Desktop**) quanto por linha de comando (**Docker CLI**).

```
+------------------+         docker push         +------------------+         docker pull         +----------------------+
|  Ambiente Local  | --------------------------> |    Docker Hub    | --------------------------> | Ambiente de Destino  |
| (Docker Desktop) |                             | (hub.docker.com) |                             | (Servidor/CI/CD/Prod)|
+------------------+                             +------------------+                             +----------------------+
```

---

## 2. Conceitos Fundamentais do Docker Hub

Antes de iniciar as operações práticas, é indispensável compreender os principais conceitos que estruturam o Docker Hub:

### 2.1 Namespaces e Nomenclatura de Repositórios

Toda imagem no Docker Hub segue uma convenção estrita de nomenclatura:

$$\text{[registry\_url/][namespace/]<nome-da-imagem>:<tag>}$$

- **Imagens Oficiais (*Official Images*)**: Não possuem namespace explícito na linha de comando (ficam sob o namespace `library/`). Exemplo: `debian:bookworm-slim`, `gcc:bookworm`, `nginx:latest`.
- **Imagens de Usuários/Organizações**: Requerem o nome de usuário ou da organização como prefixo (*namespace*). Exemplo: `meuusuario/so-simulador:1.0.0`.
- **Registry padrão**: Quando a URL do registry é omitida, o Docker assume automaticamente `docker.io` (o Docker Hub).

### 2.2 Repositórios Públicos vs. Privados

- **Público**: Qualquer pessoa pode buscar (`docker search`) e baixar (`docker pull`) a imagem sem autenticação. Ideal para projetos open source.
- **Privado**: Apenas usuários e equipes autorizados conseguem visualizar e fazer pull. O plano gratuito (*Free*) do Docker Hub inclui repositórios públicos ilimitados e 1 repositório privado gratuito (com limites de colaboradores).

### 2.3 Limites de Taxa (*Rate Limits*)

O Docker Hub aplica limites de requisições de download (*pulls*):
- **Usuários anônimos (sem login)**: 100 pulls a cada 6 horas por endereço IP.
- **Usuários autenticados (plano gratuito)**: 200 pulls a cada 6 horas por conta.
- **Planos pagos (Pro/Team/Business)**: Sem limites ou limites muito mais altos.

---

## 3. Preparação do Ambiente

Para acompanhar as instruções deste guia, certifique-se de possuir os seguintes requisitos:

1. **Conta no Docker Hub**:
   - Acesse [hub.docker.com](https://hub.docker.com) e crie uma conta gratuita.
   - Guarde seu nome de usuário (*Docker ID*), pois ele será o namespace das suas imagens.

2. **Docker Desktop Instalado**:
   - Faça o download e instale o [Docker Desktop](https://www.docker.com/products/docker-desktop/) correspondente ao seu sistema operacional (Windows com WSL2, macOS ou Linux).
   - Inicie o aplicativo e aguarde o status do motor Docker mudar para **"Engine Running"** (ícone verde no canto inferior esquerdo).

---

## 4. Autenticação e Segurança

Por boas práticas de segurança, **nunca** utilize sua senha principal da conta para fazer login no terminal ou em pipelines de automação. Em vez disso, gere um **Personal Access Token (PAT)**.

### 4.1 Criando um Personal Access Token (PAT)

1. Faça login na interface web do [Docker Hub](https://hub.docker.com);
2. Clique no seu avatar no canto superior direito e vá em **Account Settings**;
3. No menu lateral, selecione **Security** e clique em **New Access Token**;
4. Preencha a descrição (ex.: `docker-desktop-local` ou `token-estudo-registry`);
5. Defina os privilégios de acesso:
   - `Read, Write, Delete` (para uso em desenvolvimento com permissão total);
   - `Read & Write` (recomendado para desenvolvimento local e envio de imagens);
   - `Read-only` (ideal para servidores de produção que apenas baixam imagens);
6. Clique em **Generate** e **copie o token gerado imediatamente** (ele não será exibido novamente).

### 4.2 Autenticação via Docker Desktop (Interface Gráfica)

1. Abra o **Docker Desktop**;
2. No canto superior direito da janela principal, clique no botão **Sign in**;
3. Uma janela do navegador será aberta automaticamente solicitando suas credenciais do Docker Hub;
4. Após o login no navegador, o Docker Desktop conectará sua conta e exibirá seu nome de usuário no canto superior.

### 4.3 Autenticação via Docker CLI (Linha de Comando)

Para autenticar via terminal de forma segura utilizando o PAT gerado:

```bash
# Método 1: Interativo (o terminal solicitará usuário e o PAT como senha)
docker login

# Método 2: Seguro via entrada padrão (evita salvar a senha no histórico do shell)
echo "SEU_PERSONAL_ACCESS_TOKEN" | docker login -u SEU_DOCKER_ID --password-stdin
```

Se a autenticação for bem-sucedida, você verá a mensagem:
```text
Login Succeeded
```

> [!NOTE]
> As credenciais de sessão ficam salvas de forma segura no arquivo de configuração local do Docker (no Windows em `%USERPROFILE%\.docker\config.json`, integrado ao *Windows Credential Manager*).

---

## 5. Fluxo de Trabalho Prático: Build, Tag, Push e Pull

A seguir, demonstraremos o fluxo completo de publicação e consumo de uma imagem utilizando como exemplo a aplicação conteinerizada deste repositório (o simulador de tráfego `so-simulador`).

```
                +------------------------------------+
                | 1. Dockerfile + Código-fonte       |
                +------------------------------------+
                                  |
                                  v  docker build
                +------------------------------------+
                | 2. Imagem Local (so-simulador)    |
                +------------------------------------+
                                  |
                                  v  docker tag
                +------------------------------------+
                | 3. Imagem Taggeada com Namespace   |
                |    (usuario/so-simulador:1.0.0)    |
                +------------------------------------+
                                  |
                                  v  docker push
                +------------------------------------+
                | 4. Repositório no Docker Hub       |
                +------------------------------------+
                                  |
                                  v  docker pull / run
                +------------------------------------+
                | 5. Execução em Qualquer Máquina    |
                +------------------------------------+
```

---

### Passo 1: Construção da Imagem Local (*Build*)

Navegue até a raiz do projeto onde está localizado o `Dockerfile` e execute o comando de build:

```bash
# Sintaxe: docker build -t <nome-local> <caminho-do-contexto>
docker build -t so-simulador:local .
```

O Docker executará o build multi-estágio:
- **Estágio 1 (`builder`)**: compilação do código C usando a imagem `gcc:bookworm`;
- **Estágio 2 (`runner`)**: geração da imagem final enxuta baseada em `debian:bookworm-slim`.

Para verificar se a imagem foi criada com sucesso:
```bash
docker images
```

---

### Passo 2: Atribuição de Tags e Namespace (*Tag*)

Para enviar uma imagem ao Docker Hub, ela **obrigatoriamente** precisa conter o namespace (seu Docker ID) no nome.

Vamos criar duas tags para a nossa imagem:
1. Uma tag de versão semântica (`1.0.0`):
2. A tag padrão `latest` (para representar a versão mais recente):

```bash
# Substitua 'seu_usuario' pelo seu Docker ID real
docker tag so-simulador:local seu_usuario/so-simulador:1.0.0
docker tag so-simulador:local seu_usuario/so-simulador:latest
```

Ao listar as imagens novamente com `docker images`, você verá que `seu_usuario/so-simulador:1.0.0` e `seu_usuario/so-simulador:latest` compartilham o mesmo `IMAGE ID`, o que significa que são apenas apontadores para a mesma pilha de camadas.

---

### Passo 3: Publicação no Docker Hub (*Push*)

Com o Docker autenticado, publique as imagens no repositório remoto:

#### 3.1 Via Linha de Comando (CLI):

```bash
# Enviar a versão específica 1.0.0
docker push seu_usuario/so-simulador:1.0.0

# Enviar a tag latest
docker push seu_usuario/so-simulador:latest
```

**Exemplo de saída no terminal:**
```text
The push refers to repository [docker.io/seu_usuario/so-simulador]
d3a2b1c4e5f6: Pushed 
a1b2c3d4e5f6: Layer already exists 
1.0.0: digest: sha256:7c9b8e2a... size: 1152
```

Observe que camadas de imagens base que já existem nos servidores do Docker Hub recebem a indicação `Layer already exists` e não são reenviadas, economizando tempo e banda.

#### 3.2 Via Interface do Docker Desktop:

1. Abra o **Docker Desktop** e clique na aba **Images** no menu lateral esquerdo;
2. Na lista de imagens locais, localize a imagem `seu_usuario/so-simulador`;
3. Clique no menu de três pontos (**...**) ao lado da imagem e selecione **Push to Hub**;
4. O Docker Desktop exibirá uma barra de progresso do envio das camadas em tempo real.

---

### Passo 4: Inspeção no Docker Hub (Interface Web)

Após o envio, abra o navegador e acesse seu perfil no Docker Hub:
1. O repositório `seu_usuario/so-simulador` terá sido criado automaticamente (com visibilidade pública por padrão);
2. Na aba **Tags**, você poderá visualizar:
   - As tags publicadas (`1.0.0`, `latest`);
   - O tamanho compactado da imagem no registry;
   - As arquiteturas suportadas (ex.: `linux/amd64`);
   - O **Digest SHA-256** exclusivo daquela versão;
   - O histórico e detalhes de cada camada (*Layer details*).

---

### Passo 5: Consumo e Execução da Imagem (*Pull & Run*)

Para testar a portabilidade e distribuição, você pode simular a execução em outro computador (ou remover a imagem local antes de executar):

```bash
# Opcional: remover a imagem do cache local para forçar o download
docker rmi seu_usuario/so-simulador:1.0.0

# Executar a imagem diretamente a partir do Docker Hub
docker run -it --rm seu_usuario/so-simulador:1.0.0
```

**O que acontece nos bastidores:**
1. O Docker verifica se a imagem existe localmente;
2. Como não encontra, localiza o registry padrão (`docker.io`), baixa o *manifest* e as camadas necessárias da imagem remota (`Pulling from seu_usuario/so-simulador`);
3. Cria e inicia o container interativo executando a aplicação.

---

## 6. Recursos Avançados e Boas Práticas

### 6.1 Análise de Segurança com Docker Scout

O Docker Hub possui integração nativa com o **Docker Scout**, uma ferramenta de análise de composição de software (SCA) e detecção de vulnerabilidades (CVEs) em tempo de build e no registry.

Para analisar vulnerabilidades da imagem local ou publicada via CLI:

```bash
# Visão geral rápida de vulnerabilidades
docker scout quickview seu_usuario/so-simulador:1.0.0

# Relatório detalhado de CVEs encontradas nas camadas e pacotes
docker scout cves seu_usuario/so-simulador:1.0.0
```

> [!TIP]
> Usar imagens base mínimas como `debian:bookworm-slim` ou `alpine` e builds multi-estágio reduz drasticamente o número de bibliotecas desnecessárias na imagem final, diminuindo a quantidade de vulnerabilidades detectadas pelo Scout.

---

### 6.2 Estratégia de Versionamento e Gestão de Tags

Na Gerência de Configuração de artefatos conteinerizados, recomenda-se seguir uma política clara de tags:

| Estratégia | Exemplo de Tag | Quando Utilizar |
|---|---|---|
| **SemVer (Semantic Versioning)** | `1.0.0`, `1.0.1`, `2.0.0` | Releases oficiais e estáveis para produção. |
| **Commit SHA / Build ID** | `sha-a1b2c3d`, `build-452` | Deploys contínuos e rastreabilidade exata do código em CI/CD. |
| **Branch / Ambiente** | `main`, `develop`, `staging` | Ambientes de homologação e testes automatizados. |
| **Latest** | `latest` | Apenas como conveniência para uso em desenvolvimento local. |

---

### 6.3 Exemplo de Automação via CI/CD (GitHub Actions)

Um dos maiores benefícios de utilizar um Container Registry público como o Docker Hub é a facilidade de integração em pipelines de Integração Contínua e Entrega Contínua (CI/CD).

Abaixo está um exemplo de workflow do **GitHub Actions** (`.github/workflows/docker-publish.yml`) que constrói e publica a imagem automaticamente no Docker Hub a cada novo push na branch principal:

```yaml
name: Build and Push to Docker Hub

on:
  push:
    branches: [ "main" ]
    tags: [ 'v*.*.*' ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4

      - name: Configurar Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Autenticar no Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }} # Personal Access Token salvo nos Secrets

      - name: Extrair metadados e tags
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/so-simulador
          tags: |
            type=semver,pattern={{version}}
            type=sha,prefix=sha-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build e Push da Imagem
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

---

## 7. Análise Crítica: Vantagens e Limitações do Docker Hub

Para fins da pesquisa comparativa proposta neste trabalho, destacam-se os seguintes pontos sobre a plataforma:

### Vantagens
- **Adoção Universal e Padrão da Indústria**: É o registry configurado por padrão em todos os clientes Docker, sem necessidade de configuração de endpoints adicionais.
- **Catálogo de Imagens Oficiais**: Hospeda as *Docker Official Images*, que passam por curadoria de segurança, revisões constantes e documentação detalhada.
- **Interface e Usabilidade**: Integração transparente com o Docker Desktop, tornando a visualização de imagens, tags e logs intuitiva para desenvolvedores.
- **Ecossistema e Suporte**: Ampla documentação, suporte em todas as ferramentas de CI/CD e suporte nativo a builds multi-plataforma (x86_64, ARM64).

### Limitações e Desafios
- **Limites de Pull (Rate Limiting)**: O limite de 100/200 pulls a cada 6 horas pode impactar clusters Kubernetes ou pipelines de CI sem credenciais configuradas.
- **Restrições no Plano Gratuito**: Apenas 1 repositório privado gratuito e recursos limitados de controle de acesso granular (RBAC).
- **Latência e Conformidade de Dados**: Por ser um serviço SaaS global público, pode não atender a exigências estritas de privacidade ou soberania de dados de redes corporativas isoladas (onde soluções *self-hosted* como Harbor ou *cloud-native* como ECR/GAR são preferidas).

---

## 8. Conclusão

O Docker Hub desempenha um papel fundamental como o Container Registry de referência no desenvolvimento de software moderno. Compreender seu funcionamento — desde a estrutura de namespaces e tags até o gerenciamento de credenciais via PAT e automação em pipelines — fornece a base técnica e conceitual necessária para explorar soluções corporativas e privadas de registro de imagens, estabelecendo práticas sólidas de Gerência de Configuração de artefatos conteinerizados.
