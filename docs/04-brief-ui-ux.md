# Brief UI/UX — ÍRIS

| Campo | Valor |
|---|---|
| Documento | Brief UI/UX (identidade dos dashboards e relatórios) |
| Versão | 1.0 · 09/10/2026 |
| Referências | PRD ÍRIS v1.0 · documento-mestre v0.1 (§12) · Manual ARKHEN (marca, §9 identidade de dados) |

> A "interface" da ÍRIS são os **dashboards Metabase** e os **relatórios por e-mail**. Este brief fixa a identidade visual de dados; o conteúdo dos painéis está no mestre §12.

## 1. Tokens de marca (ARKHEN)

| Token | Valor | Uso |
|---|---|---|
| Azul Profundo | `#0D1B2A` | massa, cabeçalho de tabela, texto de destaque |
| Turquesa | `#00B4A6` | **acento ≤ 10%** (uma série em evidência) |
| Turquesa Claro | `#6FE9D9` | realce leve |
| Cinza Técnico | `#5A6B7A` | séries secundárias, eixos |
| Cinza Estrutura | `#E2E6EA` | grades, fundo de caixa |
| Branco | `#FFFFFF` | fundo |

Proporção **60/30/10** (turquesa é acento, nunca massa). Tipografia: Montserrat (títulos), Segoe UI/Calibri (corpo). Em botão turquesa, texto **Azul Profundo**.

## 2. Regras de gráfico (identidade de dados ARKHEN — §9)

- **Só barras horizontais e linhas.** Proibido **pizza, donut e 3D**.
- Cinza + **um** acento turquesa; **regra dos 10%**.
- **Título diz a conclusão** (não "Volume por departamento", e sim "Manutenção concentra 38% dos e-mails").
- Nunca só vermelho × verde; rótulo e eixo legíveis; sinal-ruído alto.

## 3. Dashboard executivo (P0 — mestre §12)

| Bloco | Visual |
|---|---|
| E-mails recebidos hoje | número grande (stat) |
| Volume por dia (30 d) | **linha** |
| E-mails por departamento | **barras horizontais** (ranking) |
| Prioridade (crítica/alta/média/baixa) | **4 barras horizontais** (nunca donut) |
| % SLA cumprido | número + **linha** de tendência |
| Tempo médio de resposta | número + linha |

## 4. Dashboard operacional (P0)

Recebidos, encaminhados, pendentes, resolvidos, SLA cumprido/vencido, tempo médio, responsável com maior volume, responsável com mais atrasos, solicitante mais ativo — **tabelas + barras horizontais**. Inclui a **fila de triagem humana** (e-mails com confiança < 0,6 aguardando reclassificação).

## 5. Relatórios por e-mail

**Diário (P0)** — 07:00 (America/Sao_Paulo), lista `[A CONFIRMAR PD-06]`:

```text
ÍRIS — Resumo Diário — DD/MM/AAAA
Recebidos: 152 | Concluídos: 145 | Pendentes: 7
SLA cumprido: 96% | Tempo médio: 2h13
Top assunto: Compra de Material | Top departamento: Manutenção
```

**Semanal/mensal (P1):** volume (linha), rankings (barras), SLA do período, pendências, horários de pico (heatmap — P1).

## 6. Acessibilidade

WCAG 2.1 AA no Metabase: contraste ≥ 4,5:1 para texto; não depender só de cor; rótulos sempre visíveis. Tema Metabase aplicando os tokens acima.

## 7. Multi-tenant no painel

Cada tenant vê **apenas os seus dados** — o Metabase conecta por `iris_ro`, e as consultas respeitam o `tenant_id` do cliente (RLS no banco). Nenhum painel mistura dados de clientes diferentes (RN-006).
