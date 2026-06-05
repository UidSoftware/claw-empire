# CLAUDE.md — Claw Empire (Uid Software)
> Leia este arquivo SEMPRE antes de qualquer ação.
> Última atualização: 2026-06-05

---

## O que é este projeto

**Claw Empire** é o sistema de orquestração de agents da Uid Software.
Fork de `GreenSheep01201/claw-empire` mantido em `UidSoftware/claw-empire`.

Substituiu o **Uid Office** (Claude Office Visualizer) em 2026-06-05.

**Acesso:** https://empire.uidsoftware.com.br
**API token:** em `/opt/claw-empire/.env.prod` (campo `API_AUTH_TOKEN`)

---

## Infraestrutura

```
VPS: 209.50.241.122 — Ubuntu 24.04
Projeto: /opt/claw-empire/
Container: claw-empire
Porta interna: 8005 → 8790 (app)
Nginx: empire.uidsoftware.com.br → localhost:8005
SSL: /etc/letsencrypt/live/empire.uidsoftware.com.br/ (expira 2026-09-03)
Banco: SQLite em volume Docker claw_empire_data:/app/data/claw-empire.sqlite
```

---

## Deploy e manutenção

```bash
# Ver status
docker ps | grep claw-empire
docker logs claw-empire --tail 50

# Reiniciar
cd /opt/claw-empire
docker compose -f docker-compose.prod.yml restart

# Rebuild completo
docker compose -f docker-compose.prod.yml down
docker compose -f docker-compose.prod.yml build --no-cache
docker compose -f docker-compose.prod.yml up -d

# Health check
curl http://localhost:8005/healthz
# → {"ok":true,"version":"2.0.4","app":"Claw-Empire",...}
```

---

## Variáveis de ambiente (`.env.prod`)

| Variável | Descrição |
|---|---|
| `OAUTH_ENCRYPTION_SECRET` | Encrypta tokens OAuth no SQLite — nunca alterar |
| `API_AUTH_TOKEN` | Token de acesso à API/UI — informar no primeiro acesso |
| `INBOX_WEBHOOK_SECRET` | Secret para webhooks WhatsApp/Telegram |
| `CLAUDE_CODE_OAUTH_TOKEN` | Token OAuth do Claude Code (lido pelo container) |
| `ALLOWED_ORIGINS` | `https://empire.uidsoftware.com.br` |

**Volumes montados:**
```yaml
- claw_empire_data:/app/data          # SQLite + logs (persistente)
- /opt/uid-skills:/app/skills:ro      # Skills da Uid (somente leitura)
- /root/.claude.json:/home/app/.claude.json:ro   # Credenciais Claude
```

---

## Agents da Uid configurados

| Agent | Department | Role | Skill |
|---|---|---|---|
| 🗺️ Planner ⭐ | Planning | Team Leader | PlannerSKILL |
| 🔍 Analista | Planning | Senior | AnalistaSKILL |
| 📄 doc-generator | Planning | Junior | DocGeneratorSKILL |
| 🏗️ Blueprint | Development | Team Leader | BlueprintSKILL |
| ⚒️ Forge | Development | Senior | ForgeSKILL |
| 🎨 Loom | Development | Senior | LoomSKILL |
| 🖌️ Brush | Design | Team Leader | BrushSKILL |
| 🛡️ Sentinel | QA/QC | Team Leader | SentinelSKILL |
| ✈️ Pilot | DevSecOps | Team Leader | PilotSKILL |
| 🔧 Hotfix | Operations | Senior | HotfixSKILL |

Também existem 14 agents default do Claw Empire (Luna, Pixel, Bolt, Aria, etc.)
coexistindo com os da Uid — se complementam.

**Skills carregadas:** PlannerSKILL, AnalistaSKILL, BlueprintSKILL, BrushSKILL,
ForgeSKILL, LoomSKILL, SentinelSKILL, PilotSKILL, HotfixSKILL, DocGeneratorSKILL.
Localização no container: `/app/data/custom-skills/`
Fonte: `/opt/uid-skills/.claude/skills/` (montado como volume)

---

## Claude Code — autenticação

O container usa `CLAUDE_CODE_OAUTH_TOKEN` do `.env.prod` para autenticar o
`claude` CLI instalado em `/usr/local/bin/claude` (versão 2.1.165).

O `/root/.claude.json` do host é montado em `/home/app/.claude.json:ro`
para que o sistema de detecção de providers reconheça Claude como conectado.

```bash
# Testar autenticação dentro do container
docker exec claw-empire sh -c 'claude --version'
# → 2.1.165 (Claude Code)
```

---

## Nginx

Config: `/var/www/nginx-proxy/conf.d/claw-empire.conf`
Sem `auth_basic` — autenticação feita pelo próprio Claw Empire via API token.

```bash
# Recarregar nginx
docker exec nginx-proxy-proxy-1 nginx -s reload
```

---

## Backup do Office (pré-migração)

Backup em: `/opt/backups/uid-office-20260605/`
Contém: agents/, settings.json, claude.json, CLAUDE.md

---

## Pendências pós-instalação

```
✅ Container no ar e healthy
✅ SSL empire.uidsoftware.com.br ativo
✅ Claude Code autenticado (7 agents auto-atribuídos)
✅ 10 agents da Uid criados
✅ 10 skills carregadas
✅ Planner configurado como planning leader

⬜ Configurar MCP PostgreSQL SystemD
   → Settings > MCP no Empire
   → Connection: postgresql://uid_user:***@127.0.0.1:5433/uid_sistema

⬜ Atualizar SystemD (menu Empire)
   → iframe aponta para empire.uidsoftware.com.br

⬜ Configurar GitHub OAuth
   → Settings > OAuth > GitHub
   → Permite agents fazerem push/PR

⬜ Configurar Messenger (WhatsApp/Telegram)
   → Settings > Channel Messages
   → "$" + comando → CEO Directive

⬜ Testar pipeline completo
   → CEO Desk: "Hotfix, sistema Studio Fluir. Tarefa: [desc]. Inicie."
   → Verificar Planner orquestrando Forge + Loom

⬜ Adicionar department prompt em cada department com as regras da Uid
   → Settings > Departments > Edit
```

---

## Histórico de instalação

### [2026-06-05] — Instalação inicial Claw Empire

- Backup do uid-office em `/opt/backups/uid-office-20260605/`
- Container claude-office parado e removido
- `/opt/uid-office` removido, nginx config do Office removido
- Fork `UidSoftware/claw-empire` clonado em `/opt/claw-empire`
- `.env.prod` gerado com secrets automáticos
- SSL emitido via certbot webroot
- Build Docker ~10min, container subiu healthy na porta 8005
- `auth_basic` removido do nginx (conflitava com Bearer token do frontend)
- `CLAUDE_CODE_OAUTH_TOKEN` adicionado ao `.env.prod`
- `/root/.claude.json` montado para detecção de provider
- 7 agents default auto-atribuídos ao claude pelo Empire
- 10 agents Uid criados via API
- 10 skills carregadas via `POST /api/skills/custom`
- Planner definido como planning leader

---

*Uid Software e Tecnologia LTDA — Uberlândia/MG*
*"Lead entra. Fábrica roda. Mensalidade chega."*
