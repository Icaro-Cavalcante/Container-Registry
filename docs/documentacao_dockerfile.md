# Dockerfile do Simulador de Tráfego

*Explicação técnica da containerização do projeto SO-Gerenciamento-de-Tráfego*

Repositório: [github.com/SJacksonML/SO-Gerenciamento-de-Trafego](https://github.com/SJacksonML/SO-Gerenciamento-de-Trafego)

---

## 1. Origem deste documento

Este documento foi elaborado a partir do repositório **SO-Gerenciamento-de-Trafego**, um trabalho da disciplina de Sistemas Operacionais. O código e a estrutura desse repositório foram usados como base para a criação do Dockerfile analisado aqui, desenvolvido como parte de um projeto da disciplina de **Gerência de Configuração**. Ou seja, o simulador de tráfego em si é fruto de outra disciplina, mas sua containerização — o Dockerfile detalhado nas seções seguintes — foi produzida especificamente para atender aos objetivos do trabalho de Gerência de Configuração, que trata de práticas como build reprodutível, versionamento de ambientes e empacotamento de aplicações.

## 2. Contexto do projeto

O repositório SO-Gerenciamento-de-Trafego é um trabalho prático da disciplina de Sistemas Operacionais. Ele implementa, em linguagem C, um simulador de tráfego urbano em modo texto (ASCII), usando threads POSIX (pthread), mutexes, semáforos e variáveis de condição para demonstrar conceitos como sincronização, exclusão mútua, coordenação de threads e prevenção de deadlocks.

O projeto é compilado a partir de um Makefile (alvo padrão gera o binário `bin/simulador`) e depende de um arquivo de mapa em texto (`assets/map.txt`) que descreve a malha viária simulada. Por depender de APIs POSIX (`pthread.h`, `unistd.h`), o projeto não compila nativamente em Windows/MSVC — por isso o Dockerfile é a forma recomendada de build e execução multiplataforma, junto com WSL ou GitHub Codespaces.

## 3. Visão geral do Dockerfile

O Dockerfile utiliza uma **build multi-estágio** (multi-stage build), uma técnica do Docker que permite usar uma imagem para compilar o código-fonte e outra, muito mais enxuta, apenas para executar o binário final. O resultado é uma imagem de runtime pequena, sem compilador, headers de desenvolvimento ou arquivos-fonte — apenas o executável e os recursos necessários para rodá-lo.

| Aspecto | Valor |
|---|---|
| Estágios | 2 (`builder` e `runner`) |
| Imagem de build | `gcc:bookworm` |
| Imagem de execução | `debian:bookworm-slim` |
| Binário final | `/app/bin/simulador` |
| Entrada padrão | `assets/map.txt`, 150 ticks |

## 4. Conteúdo do Dockerfile

```dockerfile
# ==========================================
# Stage 1: Build Environment
# ==========================================
FROM gcc:bookworm AS builder
WORKDIR /app
# Copy build files and source code
COPY Makefile ./
COPY include/ ./include/
COPY src/ ./src/
# Compile the application
RUN make clean && make

# ==========================================
# Stage 2: Minimal Runtime Environment
# ==========================================
FROM debian:bookworm-slim AS runner
WORKDIR /app
# Copy compiled binary and asset files
COPY --from=builder /app/bin/simulador /app/bin/simulador
COPY assets/ /app/assets/
# Execution configuration
# Default runs standard map for 150 ticks
ENTRYPOINT ["/app/bin/simulador"]
CMD ["assets/map.txt", "150"]
```

## 5. Explicação linha a linha

### 4.1 Estágio 1 — Build Environment (builder)

Este estágio é responsável apenas por compilar o código C; ele não vai parar na imagem final.

- **`FROM gcc:bookworm AS builder`** — usa a imagem oficial do gcc (baseada em Debian Bookworm) como ponto de partida, já com o compilador C, make e bibliotecas de desenvolvimento (incluindo suporte a pthread). O apelido `AS builder` permite referenciar este estágio mais adiante.
- **`WORKDIR /app`** — define `/app` como diretório de trabalho dentro do container; todos os comandos seguintes (`COPY`, `RUN`) passam a ser relativos a esse caminho.
- **`COPY Makefile ./`** · **`COPY include/ ./include/`** · **`COPY src/ ./src/`** — copiam do repositório para dentro do container apenas o necessário para compilar: o Makefile, os headers do projeto (`clock.h`, `map.h`, `render.h`, `sync.h`, `vehicle.h`) e o código-fonte (`clock.c`, `main.c`, `map.c`, `render.c`, `sync.c`, `vehicle.c`). Copiar por partes, em vez de copiar tudo de uma vez, ajuda o Docker a reaproveitar o cache de camadas quando só o código-fonte muda.
- **`RUN make clean && make`** — executa a compilação dentro do container, usando o mesmo Makefile do projeto. `make clean` garante uma build limpa (remove `obj/` e `bin/` de builds anteriores) e `make` gera o executável final em `bin/simulador`.

### 4.2 Estágio 2 — Minimal Runtime Environment (runner)

Este é o estágio que efetivamente compõe a imagem final publicada/executada.

- **`FROM debian:bookworm-slim AS runner`** — inicia um novo estágio a partir de uma imagem Debian mínima ("slim"), sem as ferramentas de compilação usadas no estágio anterior. Isso reduz bastante o tamanho final da imagem e a superfície de ataque.
- **`WORKDIR /app`** — novamente define `/app` como diretório de trabalho, agora no contexto do estágio final.
- **`COPY --from=builder /app/bin/simulador /app/bin/simulador`** — é o núcleo da build multi-estágio: copia apenas o binário já compilado do estágio `builder` para o estágio `runner`. Nenhum código-fonte, header ou ferramenta de compilação é levado para a imagem final.
- **`COPY assets/ /app/assets/`** — copia os recursos necessários em tempo de execução, como o arquivo de mapa `assets/map.txt` usado pelo simulador.
- **`ENTRYPOINT ["/app/bin/simulador"]`** — define o binário do simulador como o processo principal do container: sempre que o container roda, é esse executável que é chamado.
- **`CMD ["assets/map.txt", "150"]`** — fornece os argumentos padrão passados ao `ENTRYPOINT`: o arquivo de mapa padrão e 150 ticks de simulação. Como `CMD` é usado junto de `ENTRYPOINT`, esses valores podem ser sobrescritos ao rodar o container (por exemplo, passando outro mapa ou número de ticks) sem precisar editar o Dockerfile.

## 6. Por que usar build multi-estágio aqui

- **Imagem final menor**: a etapa de execução não carrega o gcc nem os arquivos-fonte, só o binário e os assets.
- **Build reprodutível**: a compilação sempre acontece no mesmo ambiente (`gcc:bookworm`), independente do sistema operacional de quem constrói a imagem.
- **Compatibilidade com Windows**: como o código depende de `pthread.h` e `unistd.h` (APIs POSIX), o Docker resolve a incompatibilidade nativa com Windows/MSVC citada no README do projeto.
- **Separação de responsabilidades**: fica claro o que é necessário para compilar versus o que é necessário para rodar o programa.

## 7. Como a imagem é construída e executada

De acordo com o README do repositório, o fluxo de uso é:

```bash
# Construir a imagem Docker
docker build -t so-simulador .

# Executar a simulação (interativo, com suporte a ANSI)
docker run -it --rm so-simulador

# Executar especificando mapa e quantidade de ticks (ex: 50 ticks)
docker run -it --rm so-simulador assets/map.txt 50
```

O comando `docker build` lê o Dockerfile na raiz do projeto e executa os dois estágios em sequência, gerando a imagem `so-simulador`. O `docker run` cria um container a partir dessa imagem e executa o `ENTRYPOINT`; as flags `-it` habilitam um terminal interativo (necessário porque o simulador usa códigos ANSI para atualizar a tela) e `--rm` remove o container automaticamente ao final da execução. Os argumentos passados após o nome da imagem (ex.: `assets/map.txt 50`) substituem o `CMD` padrão do Dockerfile.

## 8. Papel do `.dockerignore`

O repositório também inclui um arquivo `.dockerignore`. Ele funciona de forma semelhante a um `.gitignore`, mas para o Docker: define quais arquivos e pastas do diretório do projeto não devem ser enviados ao contexto de build (por exemplo, artefatos de build locais, como `obj/` e `bin/`, ou arquivos de relatório e documentação). Isso deixa o build mais rápido e evita copiar acidentalmente arquivos desnecessários — ou binários compilados localmente em outro sistema operacional — para dentro da imagem.

## 9. Resumo do processo de criação

Em síntese, o Dockerfile foi construído seguindo este raciocínio:

1. Identificar que o projeto é um binário C que depende de bibliotecas POSIX (pthread), inexistentes nativamente no Windows.
2. Escolher uma imagem base com toolchain completa de compilação (`gcc:bookworm`) para o estágio de build.
3. Reaproveitar o Makefile já existente no projeto (`make clean && make`) em vez de reescrever os comandos de compilação dentro do Dockerfile.
4. Isolar o resultado da compilação (o binário) de tudo o que só é necessário durante o build, usando um segundo estágio com uma imagem base mínima (`debian:bookworm-slim`).
5. Copiar para o estágio final apenas o binário e os assets (mapa) necessários para a execução, via `COPY --from=builder`.
6. Definir `ENTRYPOINT`/`CMD` para que o container já rode o simulador com parâmetros padrão sensatos (mapa padrão, 150 ticks), mas permitindo sobrescrevê-los facilmente na linha de comando.
