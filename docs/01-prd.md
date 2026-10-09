# PRD — ÍRIS

| Campo | Valor |
|---|---|
| Documento | PRD (Requisitos do Produto) |
| Projeto | ÍRIS — Secretária de Triagem de E-mail com IA · uma solução ARKHEN |
| Versão | 1.0 |
| Data | 09/10/2026 |
| Autor | Diorgi Nunes, Fundador e Diretor Executivo |
| Status | Rascunho · Em revisão |
| Últimos identificadores emitidos | RF-204 · RNF-010 · RN-009 · UC-06 · AT-06 · PD-09 |

> **Fonte da verdade:** o documento-mestre `ARKHEN_Projeto_IRIS_2026-10-08_v0.1.md` (`docs/ARKHEN_Projeto_IRIS_v0.1.md`) detalha visão, arquitetura, dados, segurança e LGPD. Este PRD é o **índice de requisitos** do Kit: fixa RF/RNF/RN/UC e aponta para a seção do mestre onde cada tema está desenvolvido. Nada é duplicado — mudou requisito aqui, reflete no mestre, e vice-versa.

## Histórico de versões

| Versão | Data | Descrição |
|---|---|---|
| 1.0 | 09/10/2026 | Primeira versão do PRD a partir do documento-mestre v0.1, com as decisões travadas: produto multi-tenant, Metabase mantido, captura por IMAP+SMTP (senha de app). |

## 0. Decisões travadas (fecham pendências do mestre)

| Pendência do mestre (§17) | Decisão | Efeito |
|---|---|---|
| PD-1/PD-2 — modelo de produto e multi-tenancy (ADR-0006) | **Produto multi-tenant** vendável a clientes | `tenant_id NOT NULL` + RLS forçado em toda tabela de negócio; ARKHEN é **operadora**, cliente é **controlador** (DPA por cliente). Ver `05-esquema-backend.md`. |
| Painel/BI | **Metabase mantido** (padrão aprovado, alinhado ao AMORA; cabe nos 8 GB) | Sem ADR de exceção. Dashboards conforme mestre §12 e `04-brief-ui-ux.md`. |
| PD-4 — captura de e-mail | **IMAP (leitura) + SMTP (encaminhamento)** via nó nativo do n8n, com **senha de app** (Gmail comum + 2FA) | Gmail API (OAuth) fica como alternativa futura registrada (ADR-0007 no TRD). |

Pendências que seguem abertas: PD-3 (provedor de IA — Vertex AI × OpenAI), PD-6 (lista/horário dos relatórios), PD-7 (retenção e RPO/RTO), PD-8 (armazenar corpo do e-mail), PD-9 (DPO nominal). Ver seção 10.

## 1. Visão geral

- **Nome:** ÍRIS.
- **Resumo em uma frase:** secretária virtual que lê cada e-mail da caixa geral, classifica (departamento, prioridade, tipo de solicitante), define o SLA, registra com protocolo e **encaminha ao responsável certo — nunca responde ao solicitante**.
- **Problema (com evidência de campo):** a triagem manual de uma caixa geral (`contato@`) atrasa o primeiro atendimento, perde prazo e não gera indicador. Detalhe no mestre §2. `[A CONFIRMAR evidência quantitativa do piloto]`
- **Público-alvo:** organizações que recebem volume relevante em caixas compartilhadas (escritórios, condomínios, sindicatos, prestadores, pequenas indústrias). Detalhe no mestre §2.
- **Plataformas:** back-of-house em Docker (n8n + PostgreSQL + Metabase + Nginx) sobre VPS Ubuntu LTS; painel Metabase via navegador. Sem app móvel nesta versão.
- **Linha ARKHEN:** AUTOMAÇÃO + DADOS.

### Atores

| ID | Ator | Descrição |
|---|---|---|
| AT-01 | Operador ARKHEN | opera a plataforma multi-tenant e implanta novos clientes |
| AT-02 | Gestor do cliente | acompanha painel, fila de triagem humana e relatórios |
| AT-03 | Responsável de setor | recebe o e-mail já encaminhado e age |
| AT-04 | Remetente externo | envia o e-mail à caixa geral (não interage com a ÍRIS) |
| AT-05 | ÍRIS (IA classificadora) | sugere classificação; não decide roteamento sozinha |
| AT-06 | Encarregado/DPO | trata direitos do titular (nominal `[A CONFIRMAR]`) |

## 2. Metas e escopo

**Metas (com prazo) — `[A CONFIRMAR números]` (mestre §2):**
- Encaminhar e-mail válido ao responsável em ≤ 2 min do recebimento.
- ≥ 95% de acerto de departamento (medido por correções manuais).
- Relatório diário entregue 100% dos dias úteis às 07:00 (America/Sao_Paulo).
- SLA cumprido ≥ 90% no mês.
- 1 tenant piloto (a própria ARKHEN) em produção; 1º cliente externo implantado em ≤ 1 dia.

**Fora do escopo desta versão:** responder ao solicitante; WhatsApp (é do AMORA); abrir ticket/OS externo; OCR de anexos; IA preventiva. Ver P1/P2 e mestre §2.

**Premissas e restrições:** pt-BR; dados no Brasil por padrão; VPS 4 vCPU / 8 GB RAM; segredos fora do código; só dado fictício em dev.

## 3. Requisitos funcionais

Prioridade: **P0 = Essencial (MVP)** · **P1 = Importante** · **P2 = Desejável**. (Alinhados ao mestre §4.)

| ID | Funcionalidade | Descrição e critério de aceite | Prioridade |
|---|---|---|---|
| RF-001 | Capturar e-mails | Ler a caixa geral por IMAP por tenant. Aceite: e-mail novo entra no fluxo em ≤ 1 min. | P0 |
| RF-002 | Classificar com IA | Departamento, prioridade, tipo de solicitante, palavras-chave, confiança. Aceite: saída JSON válida por schema. | P0 |
| RF-003 | Determinar responsável | Consultar tabela de roteamento do tenant → responsável + e-mail. Aceite: roteamento vem de dado, não de código. | P0 |
| RF-004 | Encaminhar | *Forward* ao responsável preservando o original, com guarda anti-loop. Aceite: não reenvia auto-resposta nem e-mail da própria automação. | P0 |
| RF-005 | Registrar | Protocolo único + cada mudança de estado como evento append-only. Aceite: todo e-mail gera protocolo e trilha de eventos. | P0 |
| RF-006 | Calcular SLA | SLA por prioridade a partir de `sla_regra`; marcar vencimentos (via view). Aceite: `vw_sla` correta nos casos de teste. | P0 |
| RF-007 | Dashboards Metabase | Executivo e operacional com gráficos ARKHEN (barras horizontais e linhas). Aceite: ambos publicados com usuário somente leitura. | P0 |
| RF-008 | Relatório diário | E-mail às 07:00 (America/Sao_Paulo) para lista configurável. Aceite: chega em dia útil de teste. | P0 |
| RF-009 | Triagem humana | Confiança < 0,6 → fila manual, sem encaminhar automático. Aceite: e-mail ambíguo cai na fila. | P0 |
| RF-101 | Relatórios semanal e mensal | Mesma mecânica do diário, janelas diferentes. | P1 |
| RF-102 | Alertas de SLA | Avisar responsável/coordenação em vencimento. | P1 |
| RF-103 | Sentimento | Reclamação, elogio, cobrança, informativo, urgente. | P1 |
| RF-104 | Rankings | Volume por responsável, tempo de resposta, sobrecarga. | P1 |
| RF-105 | Heatmap | Horários/dias de pico. | P1 |
| RF-201 | OCR de anexos | Extração de nº de pedido/NF/OS. | P2 |
| RF-202 | IA preventiva | Tendências de volume por assunto. | P2 |
| RF-203 | VIP/críticos | SLA diferenciado por solicitante. | P2 |
| RF-204 | Aprendizado das correções | Reclassificação assistida alimenta a regra de roteamento. | P2 |

## 4. Requisitos não funcionais (padrão ARKHEN)

| ID | Categoria | Requisito |
|---|---|---|
| RNF-001 | Segurança | Isolamento entre tenants por **RLS forçado**; teste de isolamento (tenant A × B) no CI |
| RNF-002 | Segurança | Segredos fora do código (`.env`/secret manager); TLS 1.2+/HSTS; MFA em contas admin; nada exposto além do 443 |
| RNF-003 | Privacidade | LGPD: base legal por dado, minimização, retenção configurável por tenant, direitos do titular |
| RNF-004 | Desempenho | Classificação em ≤ 8 s por e-mail; chegada→encaminhamento ≤ 2 min (p95); painel ≤ 3 s |
| RNF-005 | Disponibilidade | `[A CONFIRMAR: 99% mensal]`; RPO/RTO `[A CONFIRMAR]` |
| RNF-006 | Acessibilidade | Painel WCAG 2.1 AA; contraste ≥ 4,5:1 |
| RNF-007 | Auditoria | `eventos_email` e `lgpd_logs` imutáveis (append-only) |
| RNF-008 | Robustez de IA | Saída da IA validada por JSON Schema; conteúdo de e-mail tratado como dado (anti prompt injection) |
| RNF-009 | Custo/infra | Rodar em VPS 4 vCPU / 8 GB RAM com folga |
| RNF-010 | Backup | Dump diário dos dois bancos + volumes, **com teste de restauração** |

## 5. Regras de negócio (mestre §1, §6, §10)

| ID | Regra |
|---|---|
| RN-001 | A IA **sugere**; a regra/humano **decide**. O LLM nunca define sozinho o encaminhamento. |
| RN-002 | A ÍRIS **não responde** ao remetente em nenhuma hipótese. |
| RN-003 | Confiança < 0,6 → **triagem humana**, nunca encaminhamento automático. |
| RN-004 | Nada é apagado: toda etapa é evento no log imutável; correções geram novos eventos. |
| RN-005 | Setores, roteamento, SLA, limiar e retenção são **por tenant** (dado, não código). |
| RN-006 | Dados de um tenant nunca aparecem para outro, nem como exemplo. |
| RN-007 | Anexos: registrar metadados; conteúdo não persistido no MVP `[A CONFIRMAR PD-8]`. |
| RN-008 | SLA em minutos vem de `sla_regra`, nunca da IA. |
| RN-009 | Guarda anti-loop: descartar auto-resposta, `Auto-Submitted` e e-mail da própria automação. |

## 6. Casos de uso principais (mestre §2, §9)

| ID | Caso de uso | Ator | Fluxo principal (resumo) |
|---|---|---|---|
| UC-01 | Triagem feliz | AT-05 | e-mail → classifica (alta confiança) → encaminha → registra |
| UC-02 | Triagem humana | AT-02 | baixa confiança → fila manual → gestor reclassifica → encaminha |
| UC-03 | Acompanhar painel | AT-02 | abre Metabase → volume, fila, SLA do dia |
| UC-04 | Implantar tenant | AT-01 | cadastra setores, caixa, roteamento, limiar, retenção |
| UC-05 | Falha de envio | AT-05 | erro SMTP → evento de falha → reprocessa sem duplicar |
| UC-06 | Direito do titular | AT-06 | controlador solicita → localizar/anonimizar/eliminar registros |

## 7. Experiência do usuário

Não há jornada de usuário final (a ÍRIS roda no servidor). As "telas" são os **dashboards Metabase** e os **relatórios por e-mail**. Detalhe em `04-brief-ui-ux.md` e mestre §12.

## 8. Métricas de sucesso (mestre §2, §12)

| Métrica | Meta | Como medir |
|---|---|---|
| Acerto de departamento | ≥ 95% | correções manuais ÷ total classificado |
| Tempo chegada→encaminhamento | ≤ 2 min (p95) | eventos `recebido`→`encaminhado` |
| SLA cumprido | ≥ 90%/mês | `vw_sla` |
| Entrega do relatório diário | 100% dias úteis | log do workflow 03 |
| Tamanho da fila de triagem humana | tendência ↓ | contagem `status = triagem_humana` |

## 9. Dados e LGPD (mestre §14; detalhe em `docs/lgpd/`)

| Dado pessoal | Finalidade | Base legal | Retenção | Terceiros que recebem |
|---|---|---|---|---|
| e-mail do remetente | roteamento | legítimo interesse/execução de contrato (do controlador) | `[A CONFIRMAR: 90 dias + anonimização]` | provedor SMTP/IMAP, provedor de IA |
| assunto/corpo | classificação | idem | minimização — avaliar não guardar corpo integral (PD-8) | provedor de IA |

- **Papel da ARKHEN:** **operadora** (cliente é o controlador; DPA por cliente).
- **RIPD necessário?** **Sim** — IA tratando dado pessoal. Produzir `docs/lgpd/ripd-iris.md`.

## 10. Riscos, pendências e pronto

| ID | Pendência / risco | Dono | Prazo |
|---|---|---|---|
| PD-03 | Provedor de IA: Vertex AI (BR) × OpenAI (EUA) — custo + transferência internacional | Diorgi | `[ ]` |
| PD-06 | Lista de destinatários e horário fino dos relatórios | Diorgi | `[ ]` |
| PD-07 | Retenção, RPO/RTO, disponibilidade-alvo | Diorgi | `[ ]` |
| PD-08 | Armazenar corpo do e-mail? (metadados × trecho × integral cifrado) | Diorgi | `[ ]` |
| PD-09 | Encarregado (DPO) nominal e `encarregado@` | Diorgi | `[ ]` |

**Critério de pronto do release (cenário real):** um e-mail da caixa geral de um tenant é capturado, classificado pela ÍRIS, encaminhado ao responsável certo com guarda anti-loop, registrado com protocolo e eventos; SLA calculado por view; dashboards executivo e operacional no Metabase; relatório diário entregue às 07:00; fila de triagem humana funcionando; isolamento entre tenants testado no CI; segredos fora do código; backup diário testado; RIPD e inventário escritos.

**Aprovação:** `[nome · data]`
