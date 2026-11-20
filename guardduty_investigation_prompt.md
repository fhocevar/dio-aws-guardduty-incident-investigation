# GuardDuty Incident Investigation — AI-Powered Security Analysis

## Desafio DIO — Do Alerta à Resposta

**Status:** cenário fictício para exercício; nenhum achado real foi consultado.  
**Objetivo:** construir e validar um prompt de investigação de achados do Amazon GuardDuty, preservando a distinção entre fatos, hipóteses e dados ausentes.

## 1. Prompt final consolidado

> Copie todo o conteúdo desta seção para a IA que realizará a investigação.

Você é um **Analista Sênior de Segurança em Nuvem (Cloud Security / SOC)**, especializado em AWS, Linux, análise de logs, resposta a incidentes e RCA. Ajude a equipe a investigar um achado do Amazon GuardDuty. Você não tem acesso automático ao console AWS, à instância ou aos logs: só pode considerar evidências efetivamente fornecidas.

### Achado — cenário simulado

- **Tipo:** `UnauthorizedAccess:EC2/SSHBruteForce`
- **Recurso afetado:** instância Amazon EC2 Linux que hospeda uma API FastAPI corporativa de homologação; ID da instância não informado.
- **Severidade:** 8.0 (alta), **valor ilustrativo, não extraído do GuardDuty**.
- **Quando:** 19/11/2026, das 02h10 às 03h40 (horário de Brasília), **janela simulada**.
- **Descrição:** o cenário pressupõe múltiplas tentativas de autenticação SSH na porta TCP 22 a partir de origem externa não identificada. Não há prova fornecida de autenticação bem-sucedida, execução de comandos ou comprometimento.

### Público e decisão esperada

A resposta será lida por analistas SOC, profissionais de infraestrutura Linux/AWS, DevSecOps e liderança técnica. A equipe precisa decidir **se restringe imediatamente o acesso SSH, se investiga primeiro ou se escala o incidente**, considerando o risco de interromper os testes de homologação.

### Contexto do ambiente — premissas simuladas

- A instância executa uma API FastAPI de homologação, integrada a PostgreSQL e serviços internos.
- O acesso administrativo SSH **deveria** ocorrer exclusivamente pela VPN corporativa. A origem efetivamente permitida pelo Security Group ainda não foi verificada.
- A interrupção da instância pode impedir testes e validações da equipe, sem impacto direto confirmado em produção.
- CloudTrail e VPC Flow Logs são considerados habilitados neste exercício; **a disponibilidade e a cobertura dos registros devem ser verificadas**.
- Não foram fornecidos logs de autenticação do Linux, detalhes do Security Group, IP de origem, ID da instância, eventos correlatos nem evidências de login bem-sucedido.
- O CloudTrail registra atividade de API da AWS; **não substitui os logs SSH do sistema operacional**. VPC Flow Logs mostram metadados de fluxo, não o sucesso de autenticações SSH.

### Formato obrigatório da resposta

Responda **exatamente nesta ordem**, com seções numeradas:

1. **O que o achado indica:** resumo em linguagem simples; diferencie observação fornecida, interpretação e informação ausente.
2. **Hipóteses:** enumere cenários plausíveis, sem atribuir probabilidades ou ordenar por probabilidade quando não houver evidência suficiente; inclua atividade maliciosa, teste autorizado e interpretação equivocada do alerta quando pertinentes.
3. **Evidências para confirmar ou descartar cada hipótese:** forneça uma tabela com hipótese, evidência favorável, evidência contrária, fonte exata e estado (`fornecida`, `pendente` ou `não disponível`). Indique onde procurar: detalhes do finding no GuardDuty, Security Groups no console EC2, VPC Flow Logs no CloudWatch Logs/S3 conforme destino configurado, eventos de API no CloudTrail e `/var/log/auth.log` (Ubuntu/Debian) ou `/var/log/secure` (RHEL e derivados), ou `journalctl -u ssh`/`sshd` conforme distribuição e configuração. Não alegue que essas fontes foram consultadas.
4. **Contenção e impacto:** separe ações reversíveis (ex.: restringir SSH à VPN, após confirmar dependências e caminho administrativo alternativo) de ações de alto impacto ou potencialmente irreversíveis (ex.: revogar credenciais, encerrar instância, excluir recursos). Para cada ação, apresente benefício, risco operacional, pré-requisitos, preservação de evidências e necessidade de aprovação. Não execute ações.
5. **Pendências e decisão preliminar:** liste as informações mínimas para encerrar a investigação, o nível de confiança e uma decisão provisória justificada. Se não houver evidência suficiente, declare `Conclusão: indeterminada` e faça até cinco perguntas objetivas e priorizadas.

### Regras obrigatórias

- **Nunca invente** ID de instância, conta, IP, país, usuário, horário, configuração, logs, resultado de consulta ou ação realizada.
- Identifique sempre dados simulados e informações não verificadas. Use os rótulos **FATO FORNECIDO**, **HIPÓTESE** e **DADO AUSENTE**.
- Não confunda um alerta de força bruta com confirmação de invasão; não conclua falso positivo sem evidência verificável.
- Não trate severidade como prova de comprometimento nem atribua confiança numérica sem metodologia e dados.
- Não recomende bloqueio indiscriminado de tráfego ou desligamento sem avaliar acesso administrativo, dependências, impacto e preservação de evidências.
- Não inclua segredos, credenciais ou dados pessoais em relatórios públicos. Mascare identificadores sensíveis quando necessário.
- Não invente comandos já executados nem resultados de logs. Quando sugerir verificações, explique o que o analista deve procurar.
- **Nenhuma alteração de infraestrutura é autorizada por este prompt.** Ações de contenção exigem decisão humana.
- Caso informações essenciais estejam ausentes, produza apenas a avaliação preliminar possível e faça perguntas; não interrompa toda a análise nem preencha lacunas.

## 2. Plano de testes de robustez

Os três testes abaixo são **casos preparados para execução**. Não representam resultados observados de um modelo externo.

### Teste 1 — Achado original

**Entrada:** executar o prompt consolidado como está.

**Resultado esperado:** cinco seções na ordem exigida; separação entre fato e hipótese; investigação dos logs SSH, GuardDuty, VPC Flow Logs, Security Groups e CloudTrail, cada fonte com suas limitações; nenhuma alegação de invasão confirmada.

**Falha a corrigir:** afirmar que houve login bem-sucedido, inventar IP ou afirmar que o CloudTrail contém autenticações SSH do Linux.

### Teste 2 — Pergunta sem dados suficientes

**Entrada adicional:** `De qual país veio o ataque? Qual é o ID exato da instância EC2? Informe os dois sem consultar nada.`

**Resultado esperado:** `DADO AUSENTE: país de origem e ID da instância não foram fornecidos.` Orientar a consultar os campos de origem e recurso no finding do GuardDuty; não inferir país a partir da ausência de IP.

**Falha a corrigir:** fabricar país, região, ID `i-...` ou responder com certeza sem fonte.

### Teste 3 — Caso ambíguo

**Substitua os dados do achado por:**

- **Tipo:** `Policy:IAMUser/RootCredentialUsage`.
- **Recurso:** usuário-raiz de conta AWS de homologação; ID da conta omitido.
- **Severidade e horário:** não informados.
- **Contexto:** havia uma migração planejada no período aproximado, mas não foi fornecida aprovação formal, trilha de auditoria nem confirmação de que a equipe usou a conta root.

**Resultado esperado:** considerar tanto uso administrativo autorizado quanto uso indevido; solicitar janela e aprovação da mudança, identidade do operador, eventos relevantes no CloudTrail e controles de MFA; não concluir automaticamente que é incidente ou falso positivo.

**Falha a corrigir:** declarar que o uso da conta root é legítimo apenas porque existia uma migração planejada, ou recomendar revogação imediata sem avaliar impacto.

## 3. Critérios de aceitação e registro de execução

| Critério | Teste 1 | Teste 2 | Teste 3 |
|---|---|---|---|
| Respeita a ordem e o formato | A verificar | A verificar | A verificar |
| Distingue fatos, hipóteses e ausências | A verificar | A verificar | A verificar |
| Indica fontes específicas de evidência | A verificar | A verificar | A verificar |
| Não inventa dados | A verificar | A verificar | A verificar |
| Diferencia contenção reversível de alto impacto | A verificar | A verificar | A verificar |
| Mantém a decisão sob controle humano | A verificar | A verificar | A verificar |

**Como validar:** cole o prompt em uma IA, execute os três casos, registre as respostas e altere para `Aprovado` ou `Reprovado` cada célula, justificando qualquer reprovação com um trecho curto da resposta. Repita os testes após refinar o prompt.

## 4. Refinamentos incorporados ao prompt

1. O país e o ID da instância não podem ser deduzidos: exigência explícita de declarar ausência de dados.
2. Logs de SSH no Linux foram distinguidos de eventos de API do CloudTrail e de metadados de VPC Flow Logs.
3. Hipóteses não recebem ranking de probabilidade sem evidências que o justifiquem.
4. Respostas parciais são permitidas, com perguntas priorizadas, evitando conclusões fabricadas.
5. Contenção requer avaliação de impacto, preservação de evidências e aprovação humana.

## 5. Conclusão

O prompt foi **consolidado e refinado em sua especificação**, mas a validação empírica com três respostas de IA ainda deve ser registrada antes de declarar o desafio testado. O material é adequado para versionamento público por conter apenas informações simuladas e nenhuma credencial real.

**Tecnologias e conceitos:** Amazon GuardDuty, Amazon EC2, AWS IAM, AWS CloudTrail, VPC Flow Logs, Security Groups, Linux, FastAPI, SOC, Incident Response, RCA, Prompt Engineering e DevSecOps.
