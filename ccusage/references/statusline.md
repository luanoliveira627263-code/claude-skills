# Status line com ccusage

## Claude Code

Adicione em `~/.claude/settings.json` (ou `~/.config/claude/settings.json`). Edite o arquivo
mesclando com o que já existe — não sobrescreva outras chaves. Peça confirmação antes de
alterar configurações globais do usuário.

```json
{
  "statusLine": {
    "type": "command",
    "command": "npx -y ccusage statusline",
    "padding": 0
  }
}
```

Variantes do `command`:
- `bun x ccusage statusline` — mais rápido
- `BUN_BE_BUN=1 claude x ccusage statusline` — só no build nativo do Claude Code
- `... statusline --no-offline` — busca preços atuais (o padrão da statusline é offline)
- `... statusline --visual-burn-rate emoji` — `off|emoji|text|emoji-text`
- `... statusline --cost-source both` — `auto|ccusage|cc|both`
- `... statusline --context-low-threshold 60 --context-medium-threshold 90` (inteiros 0–255, low < medium); pode somar `"timeout": 5000`
- Outros: `--cache`, `--refresh-interval <n>`

Saída típica:
`Sonnet 4 (high) | $1.23 session / $0.06 today / $0.06 block (1h 0m left) | $0.02/hr | 50,000 (25%)`

A statusline ignora filtros de data. Se não aparecer nada: confirme que `npx`/`bun` está no
PATH do shell usado pelo Claude Code, rode o comando manualmente e verifique se há dados de uso.

## Codex (experimental)

Requer um Codex com o recurso `tui_extensions_statusline` (prerelease; o stable 0.170.0 não tem).
Em `~/.codex/config.toml` (só na config do usuário — o Codex rejeita providers de projeto):

```toml
[features]
tui_extensions_statusline = true

[mcp_servers.ccusage]
command = "/caminho/absoluto/para/ccusage"
args = ["codex", "statusline", "--mcp"]

[tui]
status_line_provider = { server = "ccusage", tool = "ccusage_codex_statusline" }
```

- Home customizado: `[mcp_servers.ccusage.env]` com `CODEX_HOME = "/caminho/absoluto"`.
- Opções extras em `args`: `--no-cost`, `--speed auto|standard|flex|fast`.
- Reinicie o Codex ou escolha "ccusage Codex usage" em `/statusline`.
- Pré-visualização no terminal: `npx ccusage codex statusline`.
