# Prompts e checklists do Kit ARKHEN — ÍRIS

## Prompt oficial por fase (genérico)

```markdown
Contexto: siga o CLAUDE.md (regras ARKHEN) e os documentos anexados
(PRD, TRD, App Flow, Brief UI/UX, Esquema do Backend, Plano de Implementação)
e o documento-mestre ARKHEN_Projeto_IRIS_...v0.1.
Tarefa: implemente somente a Fase [N] — [nome] do Plano de Implementação.
Requisitos cobertos: [RF-### ...].
Regras: não altere o que foi feito nas fases anteriores sem avisar; não adicione
bibliotecas/serviços fora do TRD; todo teste precisa ser visto falhando antes de passar;
commits pequenos e separados por natureza; um comando por vez, reporte e aguarde.
Entrega: liste os arquivos criados ou alterados, os comandos para rodar os testes
e como verificar o critério de pronto. Pare e aguarde aprovação.
```

## Prompts prontos por fase do ÍRIS

- **Fase 0 — Setup:** "...implemente somente a Fase 0 — Setup. RF cobertos: —. Entregue VPS/Docker/Nginx+TLS/UFW/`.env.example`/repo/CI; critério: HTTPS no ar e `/health` do n8n OK."
- **Fase 1 — Banco:** "...Fase 1 — Banco. RF: —. Crie bancos `iris` e `n8n`, DDL (tabelas e views do 05), índices, **RLS forçado**, usuários `iris_owner`/`iris_app`/`iris_ro`, seeds fictícios e 1 tenant. Critério: migração como `iris_owner` + **testes de isolamento multi-tenant verdes**."
- **Fase 2 — Captura + roteamento:** "...Fase 2. RF: 001,003,004,005. IMAP, guarda anti-loop, dedup por (tenant_id, message_id), roteamento, encaminhamento SMTP, registro + eventos. Critério: e-mail de teste chega ao destino e gera protocolo/eventos; auto-resposta descartada."
- **Fase 3 — IA:** "...Fase 3. RF: 002,009. Classificação, validação da saída por JSON Schema, fila de triagem humana. Critério: classifica e cai em triagem quando confiança < 0,6."
- **Fase 4 — SLA:** "...Fase 4. RF: 006. `sla_regra` por tenant, views de SLA, workflow 02. Critério: `vw_sla` correta; alerta no vencimento."
- **Fase 5 — Dashboards:** "...Fase 5. RF: 007. Metabase com `iris_ro`, tema ARKHEN, executivo + operacional por tenant. Critério: publicados; cada tenant só vê os seus dados."
- **Fase 6 — Relatório diário:** "...Fase 6. RF: 008. Workflow 03 (Cron 07:00 America/Sao_Paulo), SMTP, lista configurável. Critério: chega às 07:00 em dia útil de teste."

## Prompt de revisão (antes de codificar)

```markdown
Revise os 6 documentos + o documento-mestre contra o checklist "antes de codificar"
e o CLAUDE.md. Aponte: estados sem transição no App Flow, RF sem critério de aceite,
tabelas sem tenant_id/RLS, dados pessoais fora do inventário LGPD, serviços sem
justificativa no TRD e contradições entre documentos. Classifique cada achado como
bloqueante ou melhoria. Não reescreva nada sem aprovação.
```

## Checklist antes de codificar

- [ ] PRD com RF priorizados (P0/P1/P2), fora do escopo e métricas — **feito (v1.0)**
- [ ] TRD com stack, motivos, serviços externos, padrão de erro e regras para a IA — **feito**
- [ ] App Flow com jornada do e-mail, estados e os 4 estados de exibição — **feito**
- [ ] Brief UI/UX com tokens ARKHEN e regras de gráfico — **feito**
- [ ] Esquema com tabelas, tipos, relacionamentos, índices e **RLS** — **feito**
- [ ] Plano de implementação com fases e critérios de pronto — **feito**
- [ ] `CLAUDE.md` do repositório preenchido (estado, últimos IDs, "não construir ainda")
- [ ] Papel LGPD definido (operadora) e inventário de dados iniciado
- [ ] Pendências abertas registradas: PD-03 (IA), PD-06 (lista relatório), PD-07 (retenção/RPO/RTO), PD-08 (corpo), PD-09 (DPO)

## Checklist antes de publicar (ÍRIS)

- [ ] Todos os RF P0 testados; **testes de isolamento tenant A × tenant B passando**
- [ ] Guarda anti-loop testada (auto-resposta, `Auto-Submitted`, remetente da automação)
- [ ] Saída da IA validada por JSON Schema; conteúdo de e-mail tratado como dado (anti prompt injection)
- [ ] Só 443 público; n8n/Metabase/SSH atrás de proxy + VPN/allowlist + **MFA**
- [ ] Segredos no `.env`/secret manager; `N8N_ENCRYPTION_KEY` definida; rotação combinada
- [ ] TLS 1.2+/HSTS; criptografia em repouso dos volumes/backup
- [ ] **Backup diário dos dois bancos + volumes, com teste de restauração**
- [ ] `lgpd_logs` e `eventos_email` imutáveis; logs sem dado pessoal
- [ ] RIPD escrito (IA sobre dado pessoal) e inventário de dados completo
- [ ] Retenção/anonimização por tenant configurada; direitos do titular com canal e prazo (15 dias)
- [ ] DPA com cada cliente (operadora) — revisão jurídica
- [ ] Dashboards com gráficos ARKHEN (sem pizza/donut/3D); acessibilidade AA no painel
- [ ] Revisão jurídica feita (contrato, DPA, políticas v2.1 aplicáveis)
