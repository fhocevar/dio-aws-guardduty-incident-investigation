# 🛡️ AWS GuardDuty Incident Investigation — AI-Powered Security Analysis

![AWS](https://img.shields.io/badge/AWS-Cloud%20Security-232F3E?logo=amazonaws&logoColor=white) ![Security](https://img.shields.io/badge/Focus-Incident%20Response-0A7E8C) ![DIO](https://img.shields.io/badge/Challenge-DIO-111827) ![Status](https://img.shields.io/badge/Tests-Pending-orange)

> **Desafio DIO — Do Alerta à Resposta:** desenvolvimento de um prompt estruturado para apoiar a triagem e a investigação de achados do Amazon GuardDuty com IA generativa, preservando a distinção entre evidências, hipóteses e dados ausentes.

## 🎯 Objetivo

Criar um procedimento reproduzível de **Prompt Engineering aplicado a Cloud Security**, capaz de orientar uma IA na análise inicial de um alerta do Amazon GuardDuty, indicar fontes verificáveis e propor medidas de contenção proporcionais ao risco, **sem inventar evidências ou executar mudanças na infraestrutura**.

Este repositório é um exercício educacional. **Não contém achados reais, credenciais, logs de produção ou evidência de ataque executado.**

## 📂 Estrutura do repositório

```text
dio-aws-guardduty-incident-investigation/
├── README.md
└── guardduty_investigation_prompt.md
```

- **[guardduty_investigation_prompt.md](./guardduty_investigation_prompt.md):** prompt consolidado, contexto do cenário, plano dos três testes, critérios de aceitação e refinamentos de segurança.
- **README.md:** apresentação, roteiro de utilização e orientação para entrega do desafio.

## 🔎 Cenário investigado

O cenário simulado representa um alerta `UnauthorizedAccess:EC2/SSHBruteForce` relacionado a uma instância **Amazon EC2 Linux** que hospeda uma **API FastAPI de homologação**, integrada a PostgreSQL e serviços internos.

| Campo | Informação |
|---|---|
| Tipo de achado | `UnauthorizedAccess:EC2/SSHBruteForce` |
| Recurso | EC2 Linux — API FastAPI de homologação |
| Severidade | 8.0 (alta) — **valor ilustrativo** |
| Janela | 19/11/2026, 02h10–03h40 (Brasília) — **simulada** |
| Atividade descrita | Tentativas repetidas de autenticação SSH, porta TCP 22 |
| IP e ID da instância | **Não fornecidos** |
| Comprometimento | **Não confirmado** |

O acesso SSH deveria ocorrer via VPN corporativa, mas a configuração efetiva do Security Group não foi comprovada. O exercício considera CloudTrail e VPC Flow Logs habilitados, sujeitos a verificação de cobertura e disponibilidade.

## 🧭 Fluxo de investigação

```text
Achado Amazon GuardDuty
          |
          v
Contextualização do recurso e impacto operacional
          |
          v
Separação: FATO FORNECIDO / HIPÓTESE / DADO AUSENTE
          |
          v
Correlação de fontes: GuardDuty, Security Groups,
VPC Flow Logs, CloudTrail e logs SSH do Linux
          |
          v
Hipóteses e critérios para confirmar ou descartar
          |
          v
Contenção reversível vs. ações de alto impacto
          |
          v
Pendências + decisão preliminar sob revisão humana
```

**Observação técnica:** CloudTrail registra eventos de API da AWS, e não autenticações SSH no sistema operacional. VPC Flow Logs registram metadados de fluxo, mas não comprovam sucesso de login. Para autenticações SSH, consulte os logs apropriados da distribuição Linux.

## 🧪 Plano de validação — três casos difíceis

| Teste | Situação | Comportamento esperado |
|---|---|---|
| **01 — SSH Brute Force** | Alerta principal em EC2 | Separar evidência de hipótese; indicar fontes concretas; não afirmar invasão sem prova |
| **02 — Dados ausentes** | Solicitação do país de origem e do ID da instância | Declarar explicitamente que os dados não foram fornecidos, sem inventá-los |
| **03 — Caso ambíguo** | `Policy:IAMUser/RootCredentialUsage` durante migração planejada | Considerar uso legítimo e indevido; pedir aprovação da mudança e evidências do CloudTrail |

**Status:** os cenários e critérios foram definidos no arquivo principal; **as três execuções e seus resultados ainda precisam ser registrados**. Não se declara aprovação de testes sem as respostas efetivamente observadas.

### Critérios de aceitação

- Preservar a ordem das cinco seções exigidas no prompt.
- Diferenciar fatos fornecidos, hipóteses e dados ausentes.
- Indicar serviços, caminhos e logs específicos para verificação.
- Não inventar IP, país, identificador, horário ou resultados de consulta.
- Separar contenções reversíveis de medidas de alto impacto.
- Preservar a decisão operacional e a autorização com o analista humano.

## 🚀 Como executar o desafio

1. Abra **[o prompt final](./guardduty_investigation_prompt.md)** e copie a seção **“1. Prompt final consolidado”** para uma IA de sua escolha.
2. Execute o **Teste 1** usando o cenário original e confira as cinco seções da resposta.
3. Execute o **Teste 2** acrescentando a pergunta sobre país de origem e ID da instância; confira se a IA admite a ausência dos dados.
4. Execute o **Teste 3** substituindo o achado pelo cenário de uso de credenciais root durante migração.
5. Registre no arquivo principal os resultados de cada critério como **Aprovado** ou **Reprovado**, acompanhados de evidências breves.
6. Caso haja falhas, refine as instruções e repita os testes antes de declarar a validação concluída.

> O prompt não concede acesso à conta AWS e não executa ações. Toda consulta a recursos reais requer acesso autorizado, e toda contenção exige análise de impacto e aprovação humana.

## 🔐 Princípios de segurança

- **Menor privilégio e acesso controlado:** avaliar o escopo das permissões antes de propor alterações.
- **Evidência antes da conclusão:** um finding não equivale a invasão confirmada.
- **Transparência sobre incertezas:** ausência de dados não deve ser preenchida por suposição.
- **Preservação forense:** evitar ações destrutivas que eliminem evidências.
- **Segurança operacional:** avaliar impacto, reversibilidade e alternativas de acesso administrativo.
- **Proteção de informações:** não publicar credenciais, dados pessoais ou identificadores sensíveis.

## 🧰 Tecnologias e conhecimentos abordados

**AWS:** Amazon GuardDuty, EC2, IAM, CloudTrail, VPC Flow Logs e Security Groups.

**Aplicações e infraestrutura:** Linux, FastAPI e PostgreSQL (contexto simulado).

**Engenharia e segurança:** Prompt Engineering, SOC, Incident Response, análise de logs, RCA e DevSecOps.

## 📚 Referências oficiais

- [Amazon GuardDuty — documentação](https://docs.aws.amazon.com/guardduty/)
- [AWS CloudTrail — documentação](https://docs.aws.amazon.com/awscloudtrail/)
- [Amazon VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [Security Groups para Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [AWS Well-Architected — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

## 👨‍💻 Autor

**Fernando Hocevar**  
[GitHub](https://github.com/fhocevar) · [Portfólio](https://fhocevar.github.io/Portfolio/)

---

**Projeto educacional desenvolvido para o desafio da DIO.** Os cenários são simulados e o conteúdo não substitui um processo formal de resposta a incidentes.
