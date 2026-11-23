# Arquivo de configuração do ccusage

JSON puro (sem comentários). Comece com o schema para autocompletar:

```json
{
  "$schema": "https://ccusage.com/config-schema.json",
  "defaults": {
    "timezone": "America/Sao_Paulo",
    "offline": true
  },
  "commands": {
    "daily": { "breakdown": true },
    "blocks": { "active": true, "tokenLimit": "500000" },
    "statusline": { "refreshInterval": 5 }
  }
}
```

## Onde é procurado (o primeiro encontrado vence; não há merge)

1. `.ccusage/ccusage.json` no diretório atual
2. `$XDG_CONFIG_HOME/claude/ccusage.json` (padrão `~/.config/claude/ccusage.json`)
3. `~/.claude/ccusage.json`

Com `CLAUDE_CONFIG_DIR` definido, procura `ccusage.json` em cada diretório listado.
`--config <arquivo>` ignora a descoberta (e JSON inválido vira erro, em vez de aviso).

Precedência: **flags da CLI > arquivo > padrões**.

## Chaves

- `defaults` — vale para todos os comandos
- `commands.<relatório>` — `daily`, `weekly`, `monthly`, `session`, `blocks`, `statusline`
- Por agente: `commands."codex daily"` (ou `"codex:daily"`), ou seções `claude`, `codex`, `pi`… com seus próprios `defaults`/`commands`
- Nomes em camelCase das flags: `noCost`, `breakdown`, `timezone`, `tokenLimit`, `sessionLength`, `refreshInterval`, `startOfWeek`, `modelLabelAliases`, `pricingOverrides`
- `pi.stores` — stores extras no formato pi

Chaves desconhecidas geram aviso.

## Preços customizados

A chave é o `modelName` como aparece em `--json` → `modelBreakdowns`:

```json
{
  "pricingOverrides": {
    "meu-modelo": {
      "inputCostPerToken": 0.000003,
      "outputCostPerToken": 0.000015
    }
  }
}
```

Também aceita campos de cache e `maxInputTokens`.
