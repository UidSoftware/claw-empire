# CLAUDE.md — Claw Empire (Uid Software)
> Leia este arquivo SEMPRE antes de qualquer ação.
> Última atualização: 2026-06-08 (v3 — merge UidSkills)

---

## O que é este projeto

**Claw Empire** é o sistema de orquestração de agents da Uid Software.
Fork de `GreenSheep01201/claw-empire` mantido em `UidSoftware/claw-empire`.

Substituiu o **Uid Office** (Claude Office Visualizer) em 2026-06-05.

**Acesso:** https://empire.uidsoftware.com.br
**API token:** em `/opt/claw-empire/.env.prod` (campo `API_AUTH_TOKEN`)

---

## Como o Claude Code usa este projeto

Ao abrir o Claude Code neste diretório (ou conectar via Claw Empire), você tem acesso
ao time completo de agents da Uid. Cada skill é um especialista.

Skills disponíveis em `/opt/claw-empire/.claude/skills/` (copiadas do uid-skills).

**Nunca execute uma skill sem ler sua descrição primeiro.**
**Nunca pule etapas do pipeline — o Planner orquestra a ordem.**

---

## O Pipeline da Fábrica

```
LEAD NO BANCO (MCP PostgreSQL — vitrine_lead)
        ↓
   [PLANNER]          ← lê lead via MCP, qualifica, orquestra tudo
        ↓
   [ANALISTA]         ← elicita, modela, documenta (Sommerville)
        ↓
   [ARQUITETURA]      ← Luiz Eduardo preenche no SystemD
                         Office → Novo Projeto → Arquitetura Técnica
                         Salva em ordens_arquiteturatecnica (banco)
                         Planner lê via MCP e continua
        ↓
   [DOC-GENERATOR]    ← gera 8 documentos do projeto
        ↓
   [BLUEPRINT] + [BRUSH] ← técnica + visual (paralelo)
        ↓
   [FORGE] + [LOOM]   ← backend + frontend (paralelo)
        ↓
   [SENTINEL]         ← valida tudo — nada passa sem aprovação
        ↓
   [PILOT]            ← CI/CD, deploy na VPS, zero SSH manual
        ↓
SISTEMA EM PRODUÇÃO → MENSALIDADE NA CONTA 💰
```

**Pipeline de manutenção (Hotfix):**
```
Boss CLI (CEO Desk)
    ↓
Hotfix (lê CLAUDE.md do projeto, diagnóstico, handoff)
    ↓
Planner (classifica tarefas, orquestra)
    ↓
Forge + Loom (paralelo)
    ↓
Sentinel → Pilot
```

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
- claw_empire_data:/app/data                          # SQLite + logs (persistente)
- /opt/uid-skills:/app/skills:ro                      # Skills da Uid (somente leitura)
- /opt/claw-empire-config/claude-user.json:/home/app/.claude.json   # Auth + MCP (writable)
- /opt/claw-empire-config/claude-settings.json:/home/app/.claude/settings.json:ro
# Projetos Uid montados para acesso dos agents:
- /var/www/studio-fluir:/var/www/studio-fluir
- /root/SystemD:/home/app/projects/SytemD            # /root inacessível pelo user app
- /opt/uid-skills:/opt/uid-skills
- /opt/uidmail:/opt/uidmail
- /opt/claw-empire:/opt/claw-empire
```

**Atenção:** `/opt/claw-empire-config/claude-user.json` contém o OAuth token do Claude.
Atualizar manualmente se o token expirar (copiar do `/root/.claude.json` + adicionar `mcpServers`).

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

**Skills de suporte (usadas por outros agents):**
- AnalistaUML.md → consultar ao gerar qualquer diagrama UML

---

## Metodologia

```
Scrum  → relacionamento Uid ↔ cliente
         sprints quinzenais, backlog visível, review com cliente
         board no SystemD (menu Empire)

Kanban → execução interna dos agents
         fluxo contínuo, sem cerimônia
         visualização no Claw Empire
```

---

## Stack padrão Uid (todos os projetos)

```
Backend:  Python 3.12 + Django 5.x + DRF + SimpleJWT
Frontend: React 18 + Vite + Tailwind CSS + PWA
Banco:    PostgreSQL 16
Infra:    Docker Compose + Nginx + Gunicorn
CI/CD:    GitHub Actions (zero SSH manual)
VPS:      Ubuntu 24.04 — 209.50.241.122
```

Adaptar stack apenas se o cliente tiver tecnologia legada
ou requisito técnico específico documentado no ADR.

---

## Regras absolutas (todos os agents respeitam)

```
✅ Soft delete em todos os models — NUNCA objeto.delete()
✅ DecimalField para dinheiro — NUNCA Float
✅ Autenticação por email — NUNCA username
✅ response.data.results no frontend — NUNCA .data direto
✅ Migrations geradas no dev — NUNCA na VPS
✅ Credenciais no .env — NUNCA hardcode
✅ App 'os' proibido — usar 'ordens' com URL /api/os/
✅ LivroCaixa imutável — ReadCreateViewSet
✅ Signals com transaction.atomic() — SEMPRE
✅ CI/CD via GitHub Actions — NUNCA SSH manual no fluxo normal
✅ Testes passando antes de qualquer deploy
✅ Sentinel aprova antes do Pilot executar
✅ Brush define design system antes do Loom começar
✅ Fontes: Plus Jakarta Sans + DM Sans — NUNCA Inter/Roboto/Arial
✅ Overflow-hidden NUNCA no SistemaLayout root
```

---

## Claude Code — autenticação

O container usa `CLAUDE_CODE_OAUTH_TOKEN` do `.env.prod` para autenticar o
`claude` CLI instalado em `/usr/local/bin/claude` (versão 2.1.165).

`/opt/claw-empire-config/claude-user.json` é uma cópia do `/root/.claude.json`
com `mcpServers` integrado, montado como **writable** em `/home/app/.claude.json`.

```bash
# Testar autenticação
docker exec claw-empire sh -c 'claude --version'
# → 2.1.165 (Claude Code)

# Verificar MCP
docker exec claw-empire sh -c 'claude mcp list'
# → systemd: npx ... ✓ Connected

# Se o token expirar, regenerar o claude-user.json:
python3 -c "
import json
base = json.load(open('/root/.claude.json'))
base['mcpServers'] = {
  'systemd': {'type': 'stdio', 'command': 'npx',
    'args': ['-y', '@modelcontextprotocol/server-postgres',
             'postgresql://uid_user:Uid%402026%21ForteProd@172.19.0.2:5432/uid_sistema']}
}
json.dump(base, open('/opt/claw-empire-config/claude-user.json', 'w'), indent=2)
"
docker compose -f /opt/claw-empire/docker-compose.prod.yml restart
```

---

## MCP — SystemD PostgreSQL

```
Provider: systemd
Tipo: stdio
Comando: npx -y @modelcontextprotocol/server-postgres
Banco: uid_sistema (PostgreSQL no container sytemd-db-1)
Rede: sytemd_default (172.19.0.2:5432)
Tools disponíveis: mcp__systemd__query, mcp__systemd__list_tables,
                   mcp__systemd__describe_table, mcp__systemd__list_schemas
```

O claw-empire precisa estar na rede `sytemd_default` via
`docker network connect sytemd_default claw-empire` para alcançar o
`sytemd-db-1` (172.19.0.2).

⚠️ Essa conexão **não está em nenhum compose** — um `docker restart` a
preserva, mas uma recriação do container (`docker compose up -d --no-deps
claw-empire`, necessária após mudar mounts/env) a remove. Se o MCP parar
com erro de conexão após recriar o container, reconectar manualmente:
`docker network connect sytemd_default claw-empire`.

A connection string do MCP usa a `POSTGRES_PASSWORD` do
`/root/SystemD/.env.prod` (atualmente `devpassword123`). Se essa senha
mudar, atualizar `/opt/claw-empire-config/claude-settings.json` e
`claude-user.json` (chave `mcpServers.systemd`) — senão o `tools/call query`
falha com "password authentication failed".

**Permissões MCP** (em `/root/.claude/settings.json`):
`mcp__systemd__query`, `mcp__systemd__list_tables`, `mcp__systemd__describe_table`

**Queries principais do Planner:**
```sql
-- Novos leads aguardando qualificação
SELECT * FROM vitrine_lead WHERE convertido = false ORDER BY criado_em DESC;

-- Arquitetura Técnica salva no SystemD
SELECT * FROM ordens_arquiteturatecnica ORDER BY criado_em DESC LIMIT 1;

-- Criar OS após aprovação
INSERT INTO ordens_os (cliente_id, titulo, status) VALUES (...);

-- Marcar lead como convertido
UPDATE vitrine_lead SET convertido = true WHERE id = X;
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

## Infra VPS — portas ocupadas

| Projeto | Porta | Domínio |
|---|---|---|
| Studio Fluir | 8001 | nostudiofluir.com.br |
| SystemD | 8002 | uidsoftware.com.br |
| Mailcow HTTP | 8080 | mail.uidsoftware.com.br |
| Mailcow HTTPS | 8443 | mail.uidsoftware.com.br |
| UidMail | 8084 | uidmail.uidsoftware.com.br |
| **Claw Empire** | **8005** | empire.uidsoftware.com.br |
| **Próximo cliente** | **8003+** | a definir |

> Sempre verificar porta disponível antes de definir no docker-compose.prod.yml

---

## Projetos registrados

| Projeto | Path no container | GitHub |
|---|---|---|
| Nos Studio Fluir | `/var/www/studio-fluir` | UidSoftware/NosFluir |
| SystemD | `/home/app/projects/SytemD` | UidSoftware/SystemD |
| Claw Empire | `/opt/claw-empire` | UidSoftware/claw-empire |
| UidMail | `/opt/uidmail` | UidSoftware/Uidmail |
| UidSkills | `/opt/uid-skills` | UidSoftware/UidSkills |

> `/root/SystemD` é montado como `/home/app/projects/SytemD` porque o diretório
> `/root` não é acessível pelo usuário `app` (uid 10001) dentro do container.

---

## Como testar o pipeline (projeto fictício)

```bash
# 1. Abrir Claude Code neste diretório
claude

# 2. Simular lead qualificado
"Tenho um lead: João da Silva, Salão de Beleza Corte & Estilo,
 Uberlândia/MG. Problema: controla tudo no papel e WhatsApp.
 Quer sistema para agendamentos, clientes e financeiro.
 Planner, avalie e inicie o pipeline."

# 3. Acompanhar a esteira rodar
# Planner → Analista → doc-generator → Blueprint + Brush → Forge + Loom → ...

# 4. Validar cada output antes de avançar
# 5. Identificar onde trava e ajustar a skill correspondente
```

---

## Backup do Office (pré-migração)

Backup em: `/opt/backups/uid-office-20260605/`
Contém: agents/, settings.json, claude.json, CLAUDE.md

---

## Pendências

```
✅ Container no ar e healthy
✅ SSL empire.uidsoftware.com.br ativo
✅ Claude Code autenticado (7 agents auto-atribuídos ao claude)
✅ 10 agents da Uid criados (Planner⭐, Analista, doc-generator, Blueprint,
   Forge, Loom, Brush, Sentinel, Pilot, Hotfix)
✅ 10 skills carregadas via POST /api/skills/custom
✅ Planner configurado como planning leader
✅ MCP PostgreSQL SystemD configurado e Connected
✅ 5 projetos registrados (Nos Studio Fluir, SystemD, Claw Empire, UidMail, UidSkills)
✅ Health check corrigido: curl → node fetch (curl ausente na imagem)

⬜ Atualizar SystemD (menu Empire) → iframe aponta para empire.uidsoftware.com.br
⬜ Configurar GitHub OAuth → Settings > OAuth > GitHub
⬜ Configurar Messenger (WhatsApp/Telegram) → Settings > Channel Messages
⬜ Testar pipeline completo via CEO Desk
⬜ Adicionar department prompt com regras Uid em cada department
⬜ AnalistaSKILL — integrar com MCP PostgreSQL do SystemD (lead real)
⬜ PlannerSKILL — integrar com MCP PostgreSQL do SystemD
⬜ Criar skill de n8n (notificações automáticas)
⬜ Testar pipeline completo com projeto fictício (salão de beleza)
⬜ Criar templates por segmento (saúde, salão, agro, loja)
⬜ Versionamento das skills (tag por projeto executado com sucesso)
```

---

## Contatos Uid Software

```
WhatsApp: (34) 99134-9194
Email:    contato@uidsoftware.com.br
Site:     www.uidsoftware.com.br
GitHub:   github.com/UidSoftware
VPS:      209.50.241.122 (usuário: notuidsoftware)
```

---

## Histórico

### [2026-06-10] — Rename SytemD → SystemD + correção MCP postgres

- Repo GitHub `UidSoftware/SytemD` renomeado para `UidSoftware/SystemD`
- Diretório VPS `/root/SytemD` → `/root/SystemD`
- Recursos Docker internos mantidos como `sytemd-*` via
  `COMPOSE_PROJECT_NAME=sytemd` no `.env.prod` do SystemD (não alterar —
  contém o volume `sytemd_pgdata` com o banco `uid_sistema`)
- Bind mount do claw-empire atualizado para
  `/root/SystemD:/home/app/projects/SystemD` (path interno do container
  inalterado), container recriado para aplicar o novo mount
- MCP postgres corrigido: `claude-settings.json`/`claude-user.json` tinham
  senha desatualizada (`Uid@2026!ForteProd`); senha real é `devpassword123`
  (de `.env.prod`). `tools/call query` validado OK
- Reconectado `docker network connect sytemd_default claw-empire` (caído
  após recriação do container)

### [2026-06-08] — Merge CLAUDE.md UidSkills → Empire

- Pipeline da Fábrica + pipeline Hotfix documentados
- Stack padrão Uid adicionada
- Regras absolutas adicionadas
- Metodologia Scrum/Kanban adicionada
- Infra VPS: OfficeUid 8004 → Claw Empire 8005 (atualizado)
- MCP: queries do Planner e permissões integradas numa seção única
- Pendências das duas fontes mescladas

### [2026-06-04/05] — HotfixSKILL criada + pipeline Hotfix→Planner corrigido

- HotfixSKILL criada: papel exclusivo de diagnóstico e handoff — nunca implementa código
- Guardrail: NUNCA chamar Agent(subagent_type='hotfix') recursivamente
- BrushSKILL simplificada: removida integração com ui-ux-pro-max
- Pipeline corrigido: Hotfix→Planner (não Planner→Hotfix)
- Pipeline validado em produção no Studio Fluir: 30 melhorias, 117 testes passando

### [2026-06-05 v2] — MCP, projetos e correções pós-instalação

- MCP PostgreSQL SystemD configurado via claude-user.json + rede Docker
- Health check corrigido: curl → node fetch (curl ausente na imagem Alpine)
- 5 projetos Uid registrados via POST /api/projects
- auth_basic removido do nginx (bloqueava Bearer token do frontend React)

### [2026-06-05 v1] — Instalação inicial Claw Empire

- Backup do uid-office em /opt/backups/uid-office-20260605/
- Fork UidSoftware/claw-empire clonado, SSL emitido, container healthy na porta 8005
- 10 agents Uid criados via API, 10 skills carregadas, Planner definido como planning leader

---

*Uid Software e Tecnologia LTDA — Uberlândia/MG*
*"Lead entra. Fábrica roda. Mensalidade chega."*
