# Registro de Imagem (Container Registry)

## 🎓 Contexto acadêmico
**Universidade Federal do Cariri (UFCA)**  
**Centro de Ciência e Tecnologia (CCT)**  
**Curso:** Bacharelado em Engenharia de Software  
**Disciplina:** Gerência de Configuração - ES0015  
**Professor:** Rafael Will Macedo de Araujo  

### Integrantes
- Samuel Jackson Mesquita Lima
- Ana Aisha Tomaz de Morais
- Grazielly Bibiano do Nascimento
- Elilúcio Teixeira Félix Filho
- Icaro Cavalcante de Carvalho Pinheiro
  
---
## 📦 Container Registry
Estudo sobre **Container Registries (Registros de Imagens)** no contexto da disciplina de **Gerência de Configuração**, abordando seus fundamentos, principais soluções, autenticação, segurança, versionamento, integração com CI/CD e tendências relacionadas à segurança de imagens de containers.

O trabalho combina uma **abordagem teórica**, baseada no levantamento e análise dos principais conceitos e soluções de Container Registry, com uma **pequena demonstração prática** utilizando Docker e Docker Hub.

---

## 🎯 Objetivos

### Objetivo geral

Compreender o papel dos **Container Registries** na **Gerência de Configuração de Software**.

### Objetivos específicos

- Estudar os fundamentos de Container Registries;
- Conhecer e comparar diferentes soluções;
- Analisar aspectos de segurança, autenticação e versionamento;
- Compreender a relação entre registries e CI/CD;
- Abordar tendências relacionadas à segurança de imagens;
- Demonstrar alguns dos conceitos estudados na prática.

---

## 🔬 Metodologia

O trabalho foi desenvolvido na **modalidade teórica**, por meio de pesquisa e análise técnica sobre Container Registries.

O estudo foi organizado em:

- Fundamentação dos principais conceitos;
- Levantamento de ferramentas e soluções;
- Análise comparativa;
- Estudo de autenticação e segurança;
- Análise de versionamento;
- Investigação de tendências e desafios.

Como complemento, foi realizada uma **pequena demonstração prática** utilizando Docker e Docker Hub, relacionando os conceitos estudados ao processo de construção, versionamento e publicação de imagens.

---

## 📚 Conteúdos abordados

O trabalho aborda os seguintes temas:

- Container Registry e seu papel na Gerência de Configuração;
- Imagens, layers e manifests;
- Tags e digests;
- Padrão OCI;
- Docker Hub, GHCR, Amazon ECR, Google Artifact Registry, Harbor, Nexus e Quay;
- Autenticação e controle de acesso;
- Segurança de imagens;
- Integração com CI/CD;
- Versionamento e rastreabilidade;
- Assinatura de imagens;
- SBOM;
- Escaneamento de vulnerabilidades.

---

## 📂 Organização do repositório

```text
.
├── docs/
│   ├── container_registry/
│   │   ├── fundamentacao.md
│   │   └── levantamento_ferramentas.md
│   │
│   ├── docker_docs/
│   │   ├── documentacao_dockerfile.md
│   │   └── guia_docker_hub.md
│   │
│   ├── registry-management/
│   │   ├── analise_comparativa.md
│   │   ├── seguranca_autenticacao.md
│   │   ├── tendencias_desafios.md
│   │   └── versionamento.md
│   │
│   └── referencial_teorico.md
```

### 📁 container_registry

Apresenta a fundamentação teórica e o levantamento das principais soluções de Container Registry.

### 📁 docker_docs

Reúne a documentação relacionada ao Dockerfile e à utilização do Docker Hub no exemplo prático.

### 📁 registry-management

Concentra as análises relacionadas à Gerência de Configuração, incluindo comparação de soluções, segurança, tendências e versionamento.

---
## 🐳 Exemplo prático
A parte prática utiliza Docker e Docker Hub para demonstrar, de forma simples, o fluxo de gerenciamento de uma imagem:

Dockerfile → Build → Imagem → Tag → Push → Docker Hub → Pull → Execução

O exemplo tem caráter demonstrativo, servindo para relacionar a fundamentação teórica com a utilização de um Container Registry.

---
## 📖 Referencial Teórico
### Autenticação, Segurança e Versionamento (OCI & Docker)
- **Open Container Initiative (OCI)**. *OCI Distribution Specification - Token Authentication*. 
- **Docker Documentation**. *Registry Authentication API*. 
- **Docker Documentation**. *Image Digests and Security Concepts*. 
- **Docker Documentation**. *Core Concepts: Immutability*. 
- **ORAS (OCI Registry As Storage)**. *Reference Concepts*. 

### Ferramentas e Provedores de Registry
- **GitHub Docs**. *Publishing Docker images & Working with GitHub Container Registry*. 
- **Microsoft Learn**. *Azure Container Registry Authentication*. 
- **Amazon Web Services (AWS)**. *Amazon ECR User Guide*. 
- **Google Cloud**. *Artifact Registry Documentation*. 
- **CNCF Harbor**. *Harbor Documentation*. 

### Tendências de Segurança, Assinatura e Supply Chain (Sigstore, SBOM)
- **Sigstore**. *Cosign Quickstart & Container Signing Guide*. 
- **Sigstore**. *Cosign Repository on GitHub*. 
- **OWASP CycloneDX**. *CycloneDX Specification Overview*. 
- **OpenSSF**. *SPDX & CycloneDX Standards*. 
