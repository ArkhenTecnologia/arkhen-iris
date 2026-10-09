# Kit de Documentação de Projetos ARKHEN (v1.0 · 26/09/2026)

Modelos para documentar **qualquer projeto da ARKHEN Tecnologia** antes de pedir código à IA. Base: *Kit de Documentação para Desenvolver Apps com IA* + regras do Manual ARK-MAN-001 v1.4 (cap. 11).

## Ordem de preenchimento

| # | Arquivo | Responde | Copiar para o repositório como |
|---|---|---|---|
| 1 | `01-prd.md` | O quê e para quem | `docs/01-prd.md` |
| 2 | `02-trd.md` | Com quê (stack, APIs, padrões) | `docs/02-trd.md` |
| 3 | `03-app-flow.md` | Telas, botões e destinos | `docs/03-app-flow.md` |
| 4 | `04-brief-ui-ux.md` | Visual e estrutura de cada tela | `docs/04-brief-ui-ux.md` |
| 5 | `05-esquema-backend.md` | Autenticação, tabelas, RLS | `docs/05-esquema-backend.md` |
| 6 | `06-plano-implementacao.md` | Ordem de construção | `docs/06-plano-implementacao.md` |
| — | `07-prompts-e-checklists.md` | Prompt por fase e checklists | Uso durante o desenvolvimento |
| — | `CLAUDE.md` (pasta `02_Regras_para_Projetos`) | Regras ARKHEN + do projeto | Raiz do repositório |

## Como preencher

- Troque tudo que está entre `[colchetes]`. O que já vem escrito é **padrão ARKHEN** e só muda com justificativa (ADR).
- Nada é inventado: o que ainda não foi decidido fica marcado `[A DEFINIR]` e entra na lista de pendências (PD-##).
- IDs: RF-001, RNF-001, RN-001, UC-01, AT-01, PD-01, DA-01, ADR 0001. ID nunca é reutilizado.
- Prioridade: **P0 = Essencial (MVP)** · **P1 = Importante** · **P2 = Desejável (futuro)**.
- Cada documento tem cabeçalho de versão. Mudou requisito, tabela ou decisão: sobe a versão e registra no histórico.

## Projetos que já têm documento-mestre

Se o projeto já tem um documento-mestre (ex.: MINDUS `docs/arquitetura.md`), **ele continua sendo a fonte da verdade**. Nesses casos, os arquivos do kit viram índices que apontam para as seções do documento-mestre. Preencha só o que faltar, e nunca escreva o mesmo requisito em dois lugares.

> **ÍRIS:** o documento-mestre é `docs/ARKHEN_Projeto_IRIS_v0.1.md`. Os arquivos 01 a 07 apontam para ele.

## Como entregar à IA

1. Anexe o `CLAUDE.md` e os 6 documentos no início da conversa (ou no conhecimento do Projeto do Claude).
2. Peça **uma fase do plano por vez**, usando o prompt de `07-prompts-e-checklists.md`.
3. Teste o critério de pronto e aprove antes de liberar a próxima fase.

**Regra de ouro:** se não está escrito, a IA vai inventar.
