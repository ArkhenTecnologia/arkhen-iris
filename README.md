# ÍRIS — uma solução ARKHEN

> **Secretária de triagem de e-mail com IA.** Recebe os e-mails de uma caixa geral, classifica (departamento, prioridade, tipo de solicitante), define o SLA, registra com protocolo e **encaminha ao responsável certo**. **Não responde ao solicitante** — organiza e entrega a quem deve agir.

**Linha:** ARKHEN AUTOMAÇÃO + DADOS · **Modelo:** produto multi-tenant (SaaS) · **Status:** documentação (pré-código).

"Onde a experiência vira inteligência."

---

## O que é

ÍRIS é a camada de **entrada e distribuição** de e-mails de uma organização. Enquanto o AMORA cuida do atendimento completo (WhatsApp + e-mail, com resposta), o ÍRIS resolve o primeiro problema, de forma barata e confiável: **o e-mail certo chega à pessoa certa, sem depender de alguém lendo a caixa o dia todo.**

- A IA **sugere** o setor e um grau de confiança; a **regra de roteamento do cliente decide** o destino final.
- Tudo é **registrado num log de eventos imutável** (nada é apagado).
- E-mails de baixa confiança caem numa fila de **triagem humana**, nunca são encaminhados "no chute".

## Stack (resumo — ver `docs/02-trd.md`)

| Camada | Ferramenta |
|---|---|
| Orquestração | n8n (banco próprio isolado) |
| Entrada / saída de e-mail | IMAP + SMTP (nó nativo do n8n, senha de app) |
| Classificação | Modelo de IA via API, com saída validada |
| Persistência | PostgreSQL 16 + Row Level Security forçado |
| Painel / BI | Metabase (usuário somente leitura) |
| Borda / TLS | Nginx + Let's Encrypt |
| Empacotamento | Docker Compose |
| Hospedagem | VPS HostGator / Ubuntu Server LTS (4 vCPU / 8 GB) |

## Documentação (ler nesta ordem)

Nenhuma linha de código antes de os documentos estarem fechados (regra de engenharia ARKHEN). O documento-mestre é a **fonte da verdade**; os 6 documentos do Kit apontam para ele.

- [`docs/ARKHEN_Projeto_IRIS_v0.1.md`](docs/ARKHEN_Projeto_IRIS_v0.1.md) — **documento-mestre** (fonte da verdade)
- [`docs/00-COMO-USAR.md`](docs/00-COMO-USAR.md) — como usar o Kit
- [`docs/01-prd.md`](docs/01-prd.md) — o quê e por quê (requisitos)
- [`docs/02-trd.md`](docs/02-trd.md) — com quê (stack e decisões técnicas)
- [`docs/03-app-flow.md`](docs/03-app-flow.md) — a jornada do e-mail
- [`docs/04-brief-ui-ux.md`](docs/04-brief-ui-ux.md) — dashboards e relatórios
- [`docs/05-esquema-backend.md`](docs/05-esquema-backend.md) — banco de dados (multi-tenant + RLS)
- [`docs/06-plano-implementacao.md`](docs/06-plano-implementacao.md) — como construir, fase a fase
- [`docs/07-prompts-e-checklists.md`](docs/07-prompts-e-checklists.md) — prompt por fase e checklists

Decisões de arquitetura: [`docs/adr/`](docs/adr/) · LGPD: [`docs/lgpd/`](docs/lgpd/)

## Decisões travadas

- **Multi-tenant** (ADR-0006): `tenant_id` + RLS forçado; ARKHEN operadora, cliente controlador (DPA por cliente).
- **Metabase** mantido (padrão aprovado, alinhado ao AMORA).
- **Captura IMAP + SMTP com senha de app** (ADR-0007); Gmail API como evolução futura.

## Repositório

Organização GitHub `ArkhenTecnologia` · repositório **privado** `arkhen-iris` · branches `main` / `develop` / `feat/` / `fix/` / `chore/` / `docs/` · merge só por PR com CI verde · Conventional Commits · SemVer.

---

ARKHEN TECNOLOGIA INOVA SIMPLES (I.S.) · CNPJ 68.614.461/0001-98 · Camboriú/SC · arkhentecnologia.com.br
