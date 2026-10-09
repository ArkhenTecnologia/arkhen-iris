# Inventário de Dados — ÍRIS (LGPD)

| Campo | Valor |
|---|---|
| Documento | Inventário de dados pessoais (art. 37 LGPD) |
| Versão | 0.1 · 09/10/2026 · **rascunho** |
| Papel da ARKHEN | Operadora (cliente = controlador) |

> A preencher na Fase 8. Cada coluna com dado pessoal entra aqui com finalidade, base legal, retenção e terceiros. Não é parecer jurídico.

| Dado | Tabela/campo | Finalidade | Base legal (do controlador) | Retenção | Terceiros (suboperadores) |
|---|---|---|---|---|---|
| E-mail do remetente | `email.remetente` | roteamento ao setor | legítimo interesse / execução de contrato | `[PD-07: 90 dias + anonimização?]` | provedor IMAP/SMTP, provedor de IA |
| Assunto | `email.assunto` | classificação | idem | idem | provedor de IA |
| Corpo | `email.corpo_ref` | classificação | idem | **minimização (PD-08)** — avaliar não persistir integral | provedor de IA |
| Metadados de anexo | `email.*` / evento | triagem | idem | idem | — |
| Logs de tratamento | `lgpd_logs` | prova de conformidade | obrigação legal / legítimo interesse | 6 meses (Marco Civil) + legal | — |

**Suboperadores a contratar:** provedor de IA (`[PD-03: Vertex AI × OpenAI]` — atenção a transferência internacional se EUA), provedor de e-mail (Titan/Gmail). **A confirmar:** `[PD-07]` retenção, `[PD-08]` corpo, `[PD-09]` encarregado.
