---
name: ccusage
description: "Analisa uso de tokens e custo do Claude Code (e de outros agentes CLI como Codex, OpenCode, Gemini CLI, Copilot CLI, Amp etc.) a partir dos logs locais, usando a ferramenta ccusage (npx ccusage). Use sempre que o usuário perguntar quanto gastou, quantos tokens usou, custo por dia/semana/mês/sessão/projeto/modelo, janelas de faturamento de 5 horas (blocks), burn rate, limite de tokens, ou quiser configurar uma status line de custo no Claude Code ou no Codex — mesmo que não cite 'ccusage' pelo nome. Triggers: 'quanto gastei', 'uso de tokens', 'custo do Claude Code', 'token usage', 'billing block', 'statusline de custo', 'ccusage'."
---

# ccusage — uso de tokens e custo dos agentes CLI

[ccusage](https://github.com/ccusage/ccusage) (MIT) lê os logs **locais** do Claude Code e de
outros agentes e gera relatórios de tokens e custo. Nada é enviado para fora além de uma
telemetria anônima opcional (desligável). Preços vêm de dados LiteLLM/models.dev embutidos,
atualizados online a menos que se use `--offline`.

## Como rodar

Não precisa instalar — use um runner:

```bash
npx ccusage@latest          # = daily
bunx ccusage daily          # mais rápido, se houver bun
pnpm dlx ccusage monthly
```

Se não houver Node/bun no ambiente, diga isso ao usuário em vez de inventar números.
Os logs ficam na máquina do usuário: se você estiver em um container/nuvem sem os logs
dele (`~/.claude/projects` vazio ou ausente), avise que o relatório precisa rodar onde o
Claude Code é usado.

## Fluxo recomendado

1. Entenda a pergunta (período? projeto? modelo? sessão atual?) e escolha o comando abaixo.
2. Para **analisar** números, prefira `--json` e leia o JSON — não faça parsing da tabela.
   Para **mostrar** ao usuário, a tabela padrão (ou `--compact`) já é boa.
3. Responda com os números reais retornados e o período coberto. Mencione que o custo é
   estimado (equivalente em preço de API); em planos Pro/Max o usuário não paga isso por token.

## Comandos

| Pergunta | Comando |
|---|---|
| Gasto por dia | `npx ccusage daily` |
| Hoje | `npx ccusage daily --last 1` |
| Por semana / mês | `npx ccusage weekly` · `npx ccusage monthly` |
| Por conversa/sessão | `npx ccusage session` |
| Sessão específica | `npx ccusage claude session --id <id>` |
| Janelas de 5h (limite do plano) | `npx ccusage blocks` |
| Bloco ativo + projeção | `npx ccusage blocks --active` |
| Últimos 3 dias de blocos | `npx ccusage blocks --recent` |
| Por projeto | `npx ccusage claude daily --instances` |
| Filtrar um projeto | `npx ccusage claude daily --project <nome>` |
| Por modelo | adicione `--breakdown` |
| Só um agente | `npx ccusage <agente> <relatório>` — ex.: `ccusage claude daily`, `ccusage codex weekly`, `ccusage opencode session` |

Sem subcomando de agente, os relatórios são **unificados** (todas as fontes detectadas).
Para uma pergunta só sobre Claude Code, use `ccusage claude ...`.

## Flags principais

**Período**
- `--since YYYYMMDD` / `--until YYYYMMDD` (também aceita `YYYY-MM-DD`; inclusivo)
- `--last <n>` — últimos n períodos (não vale em session/blocks/statusline; não combina com `--since/--until/--sections`)
- `--timezone <IANA|UTC|local>` (`-z`), `--start-of-week monday|sunday`

**Saída**
- `--json` (`-j`), `--breakdown` (`-b`, por modelo), `--compact`, `--order asc|desc`
- `--no-cost` remove colunas/campos de custo
- `--sections daily,weekly,monthly,session` (com `--json`), `--by-agent` (só JSON)
- `--debug` diagnósticos no stderr

**Custo/dados**
- `--mode auto|calculate|display` — `auto` usa o `costUSD` gravado quando existe; `calculate` recalcula pelos tokens; `display` só o gravado
- `--offline` (`-O`) usa preços embutidos, sem rede
- `--config <arquivo>`

**Blocks**: `--active` (`-a`), `--recent` (`-r`), `--token-limit <n|max>`, `--session-length <horas>`

**Projetos (claude)**: `--instances` (`-i`), `--project <nome>` (`-p`), `--project-aliases "uuid=Nome"`

## Exemplos

```bash
# Custo deste mês, por modelo, em JSON
npx ccusage claude monthly --last 1 --breakdown --json

# Novembro inteiro no fuso de São Paulo
npx ccusage daily --since 20261101 --until 20261130 -z America/Sao_Paulo

# Quanto falta no bloco atual com limite = maior bloco histórico
npx ccusage blocks --active --token-limit max

# Sem rede (CI, avião)
npx ccusage daily --offline
```

## Variáveis de ambiente

- `CLAUDE_CONFIG_DIR` — diretório(s) de dados do Claude Code, separados por vírgula. Padrão: `~/.config/claude` e `~/.claude`.
- `CODEX_HOME` (padrão `~/.codex`); outras fontes: `OPENCODE_DATA_DIR`, `AMP_DATA_DIR`, `GEMINI_DATA_DIR`, `COPILOT_HOME`, `QWEN_DATA_DIR`, `KIMI_DATA_DIR`, `PI_AGENT_DIR` etc.
- `LOG_LEVEL=0` silencia avisos/títulos (útil em scripts); `NO_COLOR=1` / `FORCE_COLOR=1`.
- Telemetria: `CCUSAGE_TELEMETRY_DISABLED=1` ou `DO_NOT_TRACK=1` (ou `npx ccusage telemetry disable`).

## Status line e arquivo de configuração

- Para configurar a status line no Claude Code ou no Codex, leia `references/statusline.md`.
- Para padrões persistentes (fuso, `--offline`, `tokenLimit`, preços customizados), leia `references/config.md`.

## Problemas comuns

- **Nenhum dado**: confira se existem `.jsonl` em `~/.claude/projects/` ou `~/.config/claude/projects/`, ou defina `CLAUDE_CONFIG_DIR`.
- **Datas "erradas"**: os dias seguem o fuso local; passe `-z` explicitamente.
- **Custo diferente do esperado**: teste `--mode calculate` vs `--mode display`; sem rede, preços de modelos novos podem faltar — rode sem `--offline`.
- **Data inválida ou `since > until`**: o comando sai com erro; corrija o formato.
