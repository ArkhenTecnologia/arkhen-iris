# Plano de Implementação — ÍRIS

| Campo | Valor |
|---|---|
| Documento | Plano de Implementação |
| Versão | 1.0 · 09/10/2026 |
| Referências | PRD · TRD · App Flow · Brief UI/UX · Esquema do Backend · documento-mestre v0.1 (§16) |

> Peça **uma fase por vez** (prompt em `07-prompts-e-checklists.md`). Só avance quando o critério de pronto for testado e aprovado. O que não está na fase atual é "não construir ainda".

## Fases (específicas do ÍRIS)

| Fase | O que construir | RF cobertos | Critério de pronto | Status |
|---|---|---|---|---|
| 0. Setup | VPS Ubuntu LTS, Docker, subdomínios, Nginx+TLS (Certbot), UFW (só 443/80), `.env`/`.env.example`, repositório privado `arkhen-iris`, CI | — | Stack sobe; Nginx serve HTTPS; `/health` responde; CI verde | 🟡 artefatos prontos (`infra/`, CI); falta executar no VPS |
| 1. Banco | Bancos `iris` e `n8n`; DDL (tabelas §4), views (§5), índices; **RLS forçado**; usuários `iris_owner`/`iris_app`/`iris_ro`; seeds fictícios; 1º tenant | — | Migração roda como `iris_owner`; **testes de isolamento multi-tenant verdes** | [ ] |
| 2. Captura + roteamento | IMAP, guarda anti-loop, dedup, tabela de roteamento, encaminhamento SMTP, registro + eventos | RF-001,003,004,005,009(parcial) | E-mail de teste chega ao destino certo e gera protocolo + eventos; auto-resposta é descartada | [ ] |
| 3. IA | Classificação, validação da saída (JSON Schema), fila de triagem humana | RF-002,009 | Classifica; cai em triagem quando confiança < 0,6 ou saída inválida | [ ] |
| 4. SLA | `sla_regra` por tenant, `vw_tempo_resposta`, `vw_sla`, monitor (workflow 02) | RF-006 | `vw_sla` correta nos casos de teste; alerta dispara no vencimento | [ ] |
| 5. Dashboards | Metabase com `iris_ro`, tema ARKHEN, executivo + operacional (gráficos §12), por tenant | RF-007 | Ambos publicados; cada tenant só vê os seus dados | [ ] |
| 6. Relatório diário | Workflow 03 (Cron 07:00), SMTP, lista de destino | RF-008 | Relatório chega às 07:00 em dia útil de teste | [ ] |
| 7. P1 | Semanal/mensal, alertas de SLA, sentimento, rankings, heatmap | RF-101..105 | Critério de aceite de cada RF P1 | [ ] |
| 8. LGPD e qualidade | RIPD, inventário, retenção/anonimização por tenant, direitos do titular, revisão de segurança (OWASP/8.3), acessibilidade do painel | — | Checklist "antes de publicar" completo; RIPD e inventário escritos | [ ] |
| 9. Publicação | Backup diário **com teste de restauração**, monitoramento, documentação final, go-live do tenant piloto | — | Piloto em produção; backup restaurado em teste; monitoramento ativo | [ ] |

Cada fase: ver o teste falhar antes de contar como pronto; commits pequenos e separados por natureza; merge só por PR com CI verde.

## Critério de pronto do release

Igual ao PRD §10: e-mail de um tenant capturado → classificado → encaminhado ao responsável certo (anti-loop) → registrado com protocolo e eventos; SLA por view; dashboards Metabase por tenant; relatório diário às 07:00; triagem humana funcionando; **isolamento entre tenants testado no CI**; segredos fora do código; backup testado; RIPD e inventário prontos.

## Não construir ainda

- OCR de anexos (RF-201), IA preventiva (RF-202), VIP/críticos (RF-203), aprendizado das correções (RF-204).
- Gmail API/OAuth (fica como evolução do ADR-0007).
- Qualquer API REST pública além do necessário.

## Cronograma (referência — `[A CONFIRMAR datas]`)

| Etapa | Duração estimada | Início | Fim |
|---|---|---|---|
| Documentação (este Kit) | concluída | 08/10 | 09/10/2026 |
| Fase 0–1 (infra + banco) | 1–2 semanas | [ ] | [ ] |
| Fase 2–3 (captura + IA) | 2–3 semanas | [ ] | [ ] |
| Fase 4–6 (SLA, dashboards, relatório) | 2–3 semanas | [ ] | [ ] |
| Fase 7 (P1) | 2–3 semanas | [ ] | [ ] |
| Fase 8–9 (LGPD, qualidade, go-live) | 2 semanas | [ ] | [ ] |
