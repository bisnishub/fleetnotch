# config.json

`~/Library/Application Support/FleetNotch/config.json`. Written by the Settings window and the first-run assistant; you rarely need to
edit it by hand. **No secrets here**: keys and passwords live in the Keychain (service `FleetNotch`). Missing keys fall back to defaults.

```json
{
  "intervalo": 20,
  "usuarioSSH": "",
  "maquinas": [
    {"nome": "Studio", "curto": "Studio", "host": "local", "usuario": "", "icone": "", "visivel": true},
    {"nome": "MacBook Pro", "host": "mbp", "icone": "com.apple.macbookpro"}
  ],
  "servidores": [
    {"nome": "web-01", "host": "web-01", "usuario": "deploy", "porta": 22, "servicos": ["nginx", "docker"], "docker": true}
  ],
  "email": {"intervalo": 60, "contas": [
    {"nome": "Work", "provedor": "composio", "conta": "ca_…", "endereco": "you@example.com"},
    {"nome": "Personal", "provedor": "imap", "servidor": "imap.example.org", "porta": 993, "usuario": "me@example.org"}
  ]},
  "ia": [
    {"nome": "Pro", "ferramenta": "Claude", "fonte": "claude-perfil", "perfil": "<profile id>", "app": ["/Applications/Claude.app"],
     "perguntar": "claude-cli", "modelo": ""},
    {"nome": "Plus", "ferramenta": "ChatGPT", "fonte": "codex-sessoes", "perguntar": "codex-cli"}
  ],
  "agentes": [
    {"nome": "Atlas", "tipo": "hermes", "home": "~/.hermes", "perfil": "", "foto": "fotos/<id>.png"},
    {"nome": "Nova", "tipo": "openclaw", "porta": 18789},
    {"nome": "Scout", "tipo": "generico", "url": "http://127.0.0.1:8088/health", "rotulo": "com.example.scout", "log": "~/Library/Logs/scout.log"}
  ],
  "sincronizacao": {"agentes": "backup"},
  "modulos": {"macs": true, "servidores": true, "email": true, "ia": true, "agentes": true, "sincronizacao": true, "orca": true,
              "pergunta": true, "acoes": true, "ordem": ["topo", "macs", "servidores", "email", "orca", "sincronizacao", "pergunta"]},
  "aparencia": {"ilha": true, "todasTelas": false, "corAtencao": "#FF9F0A", "corCritico": "#FF453A", "atalho": false,
                "limites": {"cpu": [60, 85], "gpu": [60, 85], "ram": [80, 92], "temp": [85, 95], "disco": [10, 5], "ia": [80, 95]}},
  "demo": false
}
```

| Key | Meaning |
|---|---|
| `host` | Tailscale name (short or MagicDNS), an IP, an `~/.ssh/config` alias, or `local` for this Mac |
| `icone` | empty = automatic from `hw.model`; or a model type such as `com.apple.macmini` |
| `servicos` | systemd units to check with `systemctl is-active` (names only: letters, digits, `@ . : _ -`) |
| `email.intervalo` | minutes between unread counts (1 Composio tool call per account each time) |
| `fonte` | `claude-perfil` (Claude Code rate-limit cache written by `statusline.py`: `~/.claude` when `perfil` is empty, else that Orca profile), `codex-sessoes` (Codex CLI sessions) or empty |
| `perguntar` | quick-ask provider: `claude-cli`, `codex-cli`, `anthropic-api`, `openai-api` or empty |
| `limites` | `[attention, critical]`; for `disco` it is **% free** (below = alert) |
| `modulos.orca` | missing = on only if `/Applications/Orca.app` exists |
| `ordem` | drawer blocks top to bottom; neighbouring `email`, `orca`, `sincronizacao` share one row |

### How `host` is reached

Every cycle the collector reads `tailscale status` and resolves each `host`:

| `host` is… | Online status from | SSH goes to |
|---|---|---|
| `local` (or empty) | — | nothing: this Mac is read directly |
| a Tailscale name or `100.x` IP that is **also a `Host` alias in `~/.ssh/config`** | Tailscale | the **alias**, so its `HostName`, `User`, `IdentityFile` and `known_hosts` entry apply |
| a Tailscale name or `100.x` IP with no alias | Tailscale | the node's Tailscale IP |
| anything else (IP, DNS name, alias of a host outside the tailnet) | the probe itself | `host` as typed |

Only plain aliases on `Host` lines count (no wildcards); `Include` and `Match` are not followed. A user set in the app's
**SSH username** field still wins over the alias's `User`.

Keychain accounts (service `FleetNotch`): `composio`, `imap:<user>@<server>`, `uazapi:<host>`, `anthropic`, `openai`.
