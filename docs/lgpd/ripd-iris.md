# RIPD — ÍRIS (Relatório de Impacto à Proteção de Dados)

| Campo | Valor |
|---|---|
| Documento | RIPD (art. 38 LGPD) |
| Versão | 0.1 · 09/10/2026 · **rascunho / a produzir na Fase 8** |
| Motivo | IA tratando dado pessoal (conteúdo de e-mail), possivelmente sensível |

> Obrigatório antes de pôr a IA para tratar dado pessoal (regra ARKHEN §8 LGPD). Estrutura a preencher com apoio jurídico. Não é parecer jurídico.

## Estrutura a preencher
1. **Descrição do tratamento** — captura, classificação por IA, encaminhamento, registro.
2. **Necessidade e proporcionalidade** — por que a IA é necessária; minimização (não guardar corpo integral — PD-08).
3. **Dados tratados** — ver `inventario-dados.md`; risco de dado sensível no corpo.
4. **Fluxo e terceiros** — provedor de IA (`[PD-03]`; transferência internacional se EUA), IMAP/SMTP.
5. **Riscos** — vazamento, prompt injection (conteúdo é dado, não instrução), classificação errada, isolamento entre tenants.
6. **Medidas** — RLS forçado, validação de saída, segredos no `.env`, TLS, backup testado, logs sem dado pessoal, retenção/anonimização.
7. **Direitos do titular** — canal, prazo de 15 dias, confirmação de eliminação/anonimização.
8. **Conclusão e responsável** — encarregado/DPO (`[PD-09]`).
