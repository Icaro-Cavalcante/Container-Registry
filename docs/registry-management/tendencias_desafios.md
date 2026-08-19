# Tendências e Desafios — Assinatura, SBOM e Vulnerabilidades

## 1. Introdução

O cenário contemporâneo de segurança de containers é marcado por tendências que
redefinem os padrões de confiança e rastreabilidade em torno de imagens
conteinerizadas. Esta seção investiga três tendências atuais — assinatura de imagens,
geração de Software Bill of Materials (SBOM) e automação de escaneamento de
vulnerabilidades — discutindo seus fundamentos, ferramentas de referência e os
desafios que ainda impõem à adoção em larga escala.

## 2. Assinatura e Verificação de Imagens (Sigstore/Cosign)

A assinatura de imagens estabelece um modelo de confiança criptográfica que substitui
a confiança implícita em um registry específico por uma verificação explícita de que
a imagem foi produzida por uma entidade conhecida e não foi alterada após sua criação.

O **Cosign**, parte do projeto **Sigstore**, é a ferramenta de referência para essa
prática: oferece uma solução open-source e gratuita para assinatura, verificação e
timestamp de artefatos conteinerizados, eliminando a necessidade de gerenciar chaves
PKI complexas manualmente — um dos principais obstáculos históricos à adoção de
assinatura digital em pipelines de software.

**Desafio associado:** a adoção de assinatura exige mudança cultural e de processo —
pipelines precisam assinar imagens no momento do build e ambientes de execução
(Kubernetes, orquestradores) precisam ser configurados para *recusar* imagens não
assinadas, o que nem sempre é trivial em infraestruturas legadas.

## 3. Software Bill of Materials (SBOM)

O SBOM é a documentação estruturada de todas as dependências, pacotes e componentes
presentes em uma imagem. Padrões como **CycloneDX** (mantido pela OWASP) e **SPDX**
(mantido pela OpenSSF) padronizam a estrutura dessa documentação, permitindo análise
automática de vulnerabilidades conhecidas (CVEs) em nível de dependência — e não
apenas no nível da imagem como um todo.

Organizações que implementam SBOM ganham visibilidade sobre riscos de supply chain e
podem agir proativamente diante da divulgação de novas vulnerabilidades, identificando
rapidamente quais imagens em produção contêm um componente afetado.

**Desafio associado:** a geração de SBOM em cada build adiciona uma etapa ao pipeline
e exige ferramentas adicionais; além disso, manter o SBOM sincronizado com o conteúdo
real da imagem ao longo de atualizações contínuas de dependências exige automação
consistente, sob risco de o SBOM se tornar desatualizado e perder valor.

## 4. Escaneamento Automático de Vulnerabilidades

O escaneamento automático de vulnerabilidades tornou-se requisito obrigatório em
ambientes corporativos e regulados. Diferente de uma auditoria manual pontual, o
escaneamento contínuo — integrado ao pipeline de CI/CD e, idealmente, também
executado periodicamente sobre imagens já publicadas — permite identificar
vulnerabilidades recém-divulgadas em dependências que já estavam em produção.

**Desafio associado:** o volume de alertas gerados por escaneamento automático pode
ser elevado, exigindo processos de triagem e priorização (por severidade, exploração
ativa, exposição) para que a prática não se torne apenas ruído ignorado pelas equipes.

## 5. Síntese das Tendências

| Tendência | Ferramenta/Padrão de Referência | Problema que resolve | Principal desafio |
|---|---|---|---|
| Assinatura de imagens | Cosign / Sigstore | Autenticidade e integridade do artefato | Mudança de processo e enforcement na execução |
| SBOM | CycloneDX, SPDX | Visibilidade de dependências e supply chain | Manter atualizado e automatizado a cada build |
| Escaneamento de vulnerabilidades | Trivy, Amazon Inspector, ferramentas nativas de registry | Detecção de CVEs conhecidas | Volume de alertas e triagem eficiente |

## 6. Conclusão

O futuro dos container registries passa por três imperativos que se reforçam
mutuamente: implementar assinatura de imagens como garantia de autenticidade; gerar e
armazenar SBOM em formato padronizado para cada imagem; e automatizar o escaneamento
contínuo de vulnerabilidades em nível de dependência, não apenas de imagem completa.
Organizações que adotam essas práticas hoje se posicionam à frente em termos de
segurança de supply chain — um tema cada vez mais relevante diante de frameworks como
o NIST Cybersecurity Framework e o SLSA (Supply chain Levels for Software Artifacts),
que formalizam níveis crescentes de maturidade nessa direção.
