---
description: Restaura la configuración de Claude Code desde el repo GitHub PRIVADO (respalda lo existente antes; no pisa secretos locales)
---

# /config-restore — restaurar la configuración de Claude Code

Restaura la configuración básica de `~/.claude` desde el repo GitHub **privado**
del usuario (el que escribió `/config-backup`). Pensado para una instalación
nueva. Argumento opcional: nombre del repo (default `claude-config`). Uso:
`/config-restore [nombre-repo]`.

Ejecuta EN ORDEN y **párate ante cualquier fallo**:

## 1. Requisitos
- `gh auth status` logueado. Si no, dilo y para.
- `REPO="$(gh api user -q .login)/<nombre o claude-config>"`.
- Verifica que `$REPO` exista y sea **privado**. Si es público, AVISA antes de
  continuar (algo publicó config; que el usuario lo revise).

## 2. Traer el repo
Clona `$REPO` a un directorio temporal (`gh repo clone "$REPO" <tmp>`).

## 3. Respaldar lo existente ANTES de pisar nada
Si `~/.claude` ya tiene configuración, haz una copia con marca de tiempo:
`~/.claude-backup-<AAAAMMDD-HHMMSS>/` con las mismas rutas de la allowlist. Dilo.

## 4. Nunca pisar secretos locales
La restauración copia **solo** las rutas de la allowlist
(`CLAUDE.md settings.json hooks/ skills/ agents/ commands/ mods/`). **Nunca**
toca ni crea: `settings.local.json`, `.claude.json`, `~/.claves`, claves SSH,
ni nada fuera de `~/.claude`. Esos son locales de la máquina y se quedan.

## 5. Copiar
Copia las rutas del repo clonado a `~/.claude/`, fusionando (no borres lo que el
repo no trae, salvo que el usuario pida un reemplazo limpio). Tras copiar:
- `chmod 600 ~/.claude/CLAUDE.md ~/.claude/settings.json` si existen;
- `chmod 700 ~/.claude/hooks ~/.claude/skills ~/.claude/agents` si existen.

## 6. Reporte
Di qué rutas se restauraron, dónde quedó el respaldo previo, y recuerda al
usuario que **los secretos locales (tokens, claves, settings.local.json) NO se
restauran**: se recrean en la máquina nueva (p. ej. `claude` login, claves SSH).
Sugiere reiniciar Claude Code para cargar la config nueva.
