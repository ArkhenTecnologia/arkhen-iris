# ADRs — ÍRIS

As decisões de arquitetura estão consolidadas no **TRD** (`docs/02-trd.md`, seção 6) e no documento-mestre (`docs/ARKHEN_Projeto_IRIS_v0.1.md`, §3). Esta pasta guarda ADRs detalhados quando uma decisão exigir registro próprio.

| ADR | Decisão | Status |
|---|---|---|
| 0001 | Sem backend customizado; lógica no n8n + banco | Aceita |
| 0002 | n8n com banco próprio isolado | Aceita |
| 0003 | Tempo derivado de log append-only | Aceita |
| 0004 | IA só sugere; saída validada por JSON Schema | Aceita |
| 0005 | Provedor de IA (Vertex AI × OpenAI) | Proposta — PD-03 |
| 0006 | Multi-tenant (`tenant_id` + RLS forçado) | Aceita |
| 0007 | Captura por IMAP + SMTP com senha de app (Gmail API como evolução) | Aceita |

Formato de cada ADR: **Contexto → Decisão → Consequência (ganha / perde / obriga) → Status.**
