<!-- SPDX-License-Identifier: MIT -->
# config-sync

Respalda y restaura la **configuración de Claude Code** (`~/.claude`) en un
repositorio **GitHub privado** del propio usuario. Pensado para no perder tu
setup y para rehidratarlo en una máquina nueva con un comando.

Dos slash-commands:

- **`/config-backup [repo]`** — sube la config básica a un repo privado
  (`claude-config` por defecto). Allowlist explícita de qué sube, exclusión dura
  de secretos y una **puerta de escaneo que ABORTA si detecta un secreto**.
- **`/config-restore [repo]`** — en una instalación nueva, trae la config desde
  ese repo privado, respaldando antes lo existente y **sin pisar los secretos
  locales** (tokens, `settings.local.json`, claves SSH).

## Qué respalda (allowlist)
`CLAUDE.md`, `settings.json`, `hooks/`, `skills/`, `agents/`, `commands/`, `mods/`.
Lo que no esté en la lista no se sube.

## Qué NUNCA sube (y por qué)
Secretos y datos locales de la máquina: `settings.local.json`, `.claude.json`
(tokens de sesión), cualquier `.env`, `.npmrc`/`.pypirc`, claves (`*.key`,
`*.pem`, `id_rsa*`, `id_ed25519*`), códigos de recuperación, y todo lo que el
escáner de secretos marque. El repo es **privado** y se **verifica** que lo sea
antes de empujar; si es público, aborta.

## Instalar
```
/plugin marketplace add isazajuancarlos/config-sync
/plugin install config-sync@isazajuancarlos
```
(En una máquina nueva: instala el plugin y luego `/config-restore`.)

## Seguridad
- **Allowlist, no denylist** (lo que no se nombra, no se sube).
- **Puerta de secretos bloqueante** antes de cada push (gitleaks/trufflehog, o un
  grep de patrones conservador como fallback).
- **Repo privado obligatorio** (se verifica; nunca config a un repo público).
- La restauración **no toca secretos locales** y respalda lo existente primero.

## Licencia
MIT © 2026 Juan Carlos Isaza.
