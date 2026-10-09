# CLAUDE.md — ÍRIS (ARKHEN Tecnologia)

> ⚠️ **Repositório substituído em 09/10/2026.** A ÍRIS foi redesenhada após a conversa com o cliente e passou a ser o **módulo de e-mail do AMORA**. A documentação vigente (versão 2.0) e o código ficam em **`ArkhenTecnologia/arkhen-crm-template`**, pasta `docs/modulos/iris/`. Este repositório é mantido só como histórico da versão 1.0 — não abra trabalho novo aqui.

> Regras ARKHEN (resumo do Manual ARK-MAN-001 v1.4) + regras do projeto ÍRIS. Leia antes de qualquer tarefa no repositório `arkhen-iris`. A versão completa e sempre vigente do manual vive no Projeto Arkhen_ÍRIS; aqui fica o essencial para codar.

## Fonte da verdade

Ordem de prevalência: **lei vigente > contrato com o cliente > documento-mestre `docs/ARKHEN_Projeto_IRIS_v0.1.md` > os 6 documentos em `docs/` > Manual da Marca > Manual Corporativo ARKHEN v1.4 > preferência do projeto.** Na divergência, o nível superior vence. **Nenhum código sem requisito documentado** (RF/RNF/RN). Todo delta é escrito a partir do documento, nunca da memória, e gera bump de versão.

## O que o ÍRIS é (e o que não é)

- **É:** secretária de triagem de e-mail. Lê, classifica, registra e **encaminha** ao setor certo.
- **Não é:** não responde ao remetente, não é CRM, não gerencia conversas. (Se o cliente precisa responder, o produto é o AMORA.)
- Produto **multi-tenant**: vários clientes na mesma instância, isolados por `tenant_id` + RLS.

## Invioláveis de engenharia (ARKHEN)

1. **A IA sugere; a regra/humano decide.** Classificação do LLM é sugestão com confiança; roteamento final é decidido por regra determinística por tenant. Saída de IA **sempre validada** (JSON Schema), nunca aplicada crua. Confiança < 0,6 → triagem humana.
2. **Multi-tenant de verdade:** `tenant_id NOT NULL` + **RLS forçado** em toda tabela de negócio, sem política permissiva, com teste de isolamento (conta A × conta B) no CI. Nunca confie só em filtro na aplicação.
3. **Tempo deriva do log de eventos** `append-only` e imutável. Nenhum campo de duração; nada aberto é apagado; cancelamentos usam catálogo de motivos.
4. **Transição de estado não prevista é recusada pela entidade**, antes de gravar.
5. **Variação de cliente é dado, nunca `if` no código.** Setores, roteamento, SLA, limiar e retenção são configuráveis por tenant. Nada hardcoded.
6. **Domínio puro:** regras de classificação e roteamento rodam e são testadas sem banco, sem rede e sem IA. Integrações externas (IMAP, SMTP, LLM) passam por camada anticorrupção (porta + adaptador + `vinculo_externo`).
7. **Erros de domínio tipados.** Domínio, testes e comentários em português; estados e eventos em `snake_case`, iguais ao DDL.
8. **Segredos nunca no código nem no front.** `.env` no `.gitignore` ou secret manager; rotação periódica. Vazou → revogar.
9. **Testes:** todo teste precisa ser visto falhando antes de contar. "Instalar de verdade" antes de cada release.
10. **Segurança:** só 443 público; n8n/Metabase/SSH atrás de Nginx + VPN/allowlist + MFA; TLS 1.2+/HSTS; conteúdo de e-mail é dado, não instrução (anti prompt injection); backup diário com teste de restauração.

## IDs e versionamento

Prefixos: `RF-###`, `RNF-###`, `RN-###`, `DA-##`, `DD-##`, `UC-##`, `PD-##`, `ADR-####`.
Mantenha a linha "últimos identificadores emitidos" no fim de `docs/01-prd.md`.

## Fluxo de trabalho com IA de código

Uma fase por vez (ver `docs/06-plano-implementacao.md`), com o prompt oficial de `docs/07-prompts-e-checklists.md`:

> **Contexto** → **Tarefa: somente a Fase N** → **RF cobertos** → **Regras** → **Entrega** → **pare e aguarde aprovação.**

Decidir no chat → executar **um comando por vez** → reportar → aprovar → seguir. Antes de qualquer commit: `pwd` + `git status`. Commits pequenos e separados por natureza (doc sem código, domínio sem infra). Merge só por PR com CI verde. Conventional Commits; SemVer.

## LGPD (papel da ARKHEN)

No ÍRIS a ARKHEN é **operadora**; cada cliente é o **controlador** e precisa de DPA. Conteúdo de e-mail é **dado pessoal** (às vezes sensível). Inventário em `docs/lgpd/inventario-dados.md`; RIPD em `docs/lgpd/ripd-iris.md`. Retenção configurável por tenant. Nada de dado real em desenvolvimento. Direitos do titular: resposta em até 15 dias. Registros de acesso: 6 meses (Marco Civil).

## Marca (quando gerar painel, relatório ou tela)

Paleta: Azul Profundo `#0D1B2A` · Turquesa `#00B4A6` · Turquesa Claro `#6FE9D9` · Cinza Técnico `#5A6B7A` · Cinza Estrutura `#E2E6EA` · Branco. Proporção 60/30/10 (turquesa é acento). Em botão turquesa, texto **Azul Profundo**, nunca branco. Gráficos: barras horizontais e linhas, **nunca pizza, donut nem 3D**; título diz a conclusão. Tipografia: Montserrat (títulos), Segoe UI/Calibri (corpo).

## Identidade (use exato)

ARKHEN TECNOLOGIA INOVA SIMPLES (I.S.) · CNPJ 68.614.461/0001-98 · Camboriú/SC · Fundador e Diretor Executivo: Diorgi Nunes · (47) 99162-2326 · diorginunes@arkhentecnologia.com.br. Nunca "LTDA", nunca "KINEXUS".

## Regras do projeto — ÍRIS (estado atual)

- **Repositório:** org `ArkhenTecnologia`, privado, `arkhen-iris`.
- **Decisões travadas:**
  - **Multi-tenant (ADR-0006, aceita):** produto vendável; `tenant_id` + RLS forçado; ARKHEN operadora, cliente controlador (DPA por cliente).
  - **Metabase mantido:** padrão aprovado (AMORA); cabe no VPS de 8 GB; sem ADR de exceção.
  - **Captura IMAP + SMTP com senha de app (ADR-0007, aceita):** Gmail/Titan comum + 2FA; Gmail API (OAuth) como evolução futura.
- **Stack:** n8n (+ Postgres isolado) · PostgreSQL 16 (`iris`) + RLS · Metabase (`iris_ro`) · Nginx 443/TLS · Docker · VPS HostGator/Ubuntu LTS, 4 vCPU/8 GB.
- **Últimos identificadores emitidos:** RF-204 · RNF-010 · RN-009 · UC-06 · AT-06 · PD-09 · ADR-0007.
- **Pendências abertas:** PD-03 (provedor de IA: Vertex AI × OpenAI) · PD-06 (lista/horário dos relatórios) · PD-07 (retenção, RPO/RTO) · PD-08 (armazenar corpo do e-mail?) · PD-09 (DPO nominal e `encarregado@`). Confirmar também: VPS HostGator dá root + Docker + datacenter BR.
- **Fase atual do Plano:** documentação concluída (Kit v1.0). Próxima: **Fase 0 — Setup** na HostGator.
- **Não construir ainda:** OCR de anexos (RF-201), IA preventiva (RF-202), VIP/críticos (RF-203), aprendizado das correções (RF-204), Gmail API/OAuth.
