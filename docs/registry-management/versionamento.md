# Versionamento de Imagens (Tags e Digests)

## 1. Introdução

O versionamento de imagens de containers representa um dos desafios mais críticos no
gerenciamento de artefatos conteinerizados. Diferente de pacotes de software
tradicionais, uma imagem de container pode ser referenciada de formas distintas — por
tag ou por digest —, cada uma com implicações diretas sobre reprodutibilidade,
segurança e confiabilidade em pipelines de CI/CD. Esta seção investiga os dois
mecanismos e consolida boas práticas de nomenclatura e gerenciamento de versões.

## 2. Tags

Uma **tag** é uma referência legível por humanos associada a uma versão de uma imagem
(por exemplo, `minha-app:1.2.0`, `minha-app:latest`, `minha-app:staging`). Tags
funcionam como ponteiros mutáveis: podem ser reatribuídas a qualquer momento, passando
a apontar para um conteúdo de imagem completamente diferente do original.

Essa mutabilidade é conveniente para fins de organização e comunicação entre equipes,
mas introduz um risco relevante em ambientes de produção: um deploy realizado hoje com
a tag `v1.0` não garante que a mesma imagem será utilizada amanhã, caso essa tag seja
sobrescrita por um novo push. Tags como `latest` e `master` são especialmente
propensas a esse problema, pois costumam ser atualizadas continuamente pelo próprio
fluxo de build.

## 3. Digests

Um **digest** é um hash criptográfico determinístico (SHA-256) calculado sobre o
conteúdo do manifest da imagem. Diferente da tag, o digest é **imutável**: qualquer
alteração no conteúdo da imagem resulta em um digest diferente, o que garante
integridade, rastreabilidade e previsibilidade.

Uma imagem referenciada por seu digest (por exemplo,
`minha-app@sha256:abcd1234...`) nunca pode ser alterada silenciosamente — é a
referência recomendada para cenários em que a reprodutibilidade exata é essencial,
como ambientes de produção regulados ou auditáveis.

## 4. Tags vs. Digests — Comparação

| Critério | Tag | Digest |
|---|---|---|
| Legibilidade | Alta (rótulo humano) | Baixa (hash SHA-256) |
| Mutabilidade | Mutável (pode ser sobrescrita) | Imutável |
| Uso recomendado | Desenvolvimento, comunicação entre equipes | Produção, auditoria, reprodutibilidade |
| Rastreabilidade | Depende de disciplina da equipe | Garantida criptograficamente |

## 5. Infraestrutura Imutável e Reprodutibilidade

A adoção de digests está diretamente ligada ao conceito de **infraestrutura
imutável**, essencial para pipelines de CI/CD que dependem de reprodutibilidade: ao
fixar o digest exato utilizado em um deployment, elimina-se a possibilidade de que uma
tag reaproveitada introduza uma imagem inesperada em produção — um requisito central
da Gerência de Configuração.

## 6. Boas Práticas de Nomenclatura e Gerenciamento

Com base na pesquisa realizada, recomenda-se:

1. **Construir imagens com tags versionadas semânticas (SemVer)**, evitando `latest`
   ou `master` como tag de deploy em produção;
2. **Armazenar metadados de deployment** que incluam o digest exato utilizado,
   permitindo auditoria e rollback preciso;
3. **Referenciar imagens por digest** em manifests de produção (Kubernetes,
   docker-compose) sempre que a reprodutibilidade for crítica;
4. **Implementar verificação de assinatura de digest**, garantindo autenticidade além
   da imutabilidade (tema aprofundado em `docs/tendencias_desafios.md`);
5. **Adotar ferramentas compatíveis com OCI Registry As Storage (ORAS)**, que suportam
   nativamente o modelo de referência por digest.

## 7. Considerações Finais

Tags e digests não são mecanismos concorrentes, mas complementares: tags facilitam a
comunicação humana e a organização do ciclo de desenvolvimento, enquanto digests
garantem a integridade e a reprodutibilidade exigidas em ambientes críticos. A
maturidade de uma estratégia de versionamento está em saber quando usar cada um —
tags para agilidade no dia a dia, digests como referência de verdade em produção.