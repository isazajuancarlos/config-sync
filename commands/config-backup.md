---
description: Respalda la configuración de Claude Code a un repo GitHub PRIVADO (allowlist + escaneo de secretos; nunca sube secretos)
---

# /config-backup — respaldar la configuración de Claude Code

Respalda la configuración **básica** de `~/.claude` a un repositorio GitHub
**privado** del usuario. Argumento opcional: el nombre del repo (default
`claude-config`). Uso: `/config-backup [nombre-repo]`.

Ejecuta estos pasos EN ORDEN y **PÁRATE ante cualquier fallo** (no sigas, no
empujes):

## 1. Requisitos
- `gh auth status` debe estar logueado. Si no, dilo y para.
- Toma el nombre del repo del argumento, o usa `claude-config`.
- `REPO="$(gh api user -q .login)/<nombre>"`.

## 2. Preparar el área de ensayo (SOLO lo de la allowlist)
Crea un directorio temporal vacío y copia **únicamente** estas rutas desde
`~/.claude` (ALLOWLIST — lo que no esté aquí NO se respalda, D21):

```
CLAUDE.md
settings.json
hooks/
skills/
agents/
commands/
mods/
```

Copia solo las que existan. **Omite enlaces simbólicos** (cópialos como lo que
apuntan solo si apuntan dentro de `~/.claude`; si no, sáltalos).

## 3. Excluir secretos SIEMPRE (defensa en profundidad)
Aunque no estén en la allowlist, borra del área de ensayo cualquier coincidencia
(por si un hook/skill los arrastró):

```
settings.local.json   *.local.json   .claude.json   .credentials.json
**/.env   **/*.env*   **/.npmrc   **/.pypirc   **/credentials*
**/*token*   **/*secret*   **/*seed*   **/*.key   **/*.pem
**/id_rsa*   **/id_ed25519*   **/*ecovery*ode*   **/Backup-codes*
```

## 4. PUERTA de secretos (bloqueante) — antes de cualquier push
Escanea el área de ensayo. Usa lo que haya, en este orden:
- `gitleaks detect --no-git --source <stage> --redact -v` si está instalado, o
- `trufflehog filesystem <stage>` si está, o
- (fallback) `grep -rIEn '(gh[opsu]_[A-Za-z0-9]{36}|AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]*PRIVATE KEY-----|xox[baprs]-|sk-[A-Za-z0-9]{20,})' <stage>`.

**Si el escaneo reporta UN SOLO hallazgo: ABORTA.** No empujes. Enséñale al
usuario el hallazgo (redactado) y qué archivo lo trajo, y para. El respaldo que
sube un secreto es peor que no respaldar (Ley 1581 / credenciales).

## 5. Repo privado (verificar/crear PRIVADO)
- Si `gh repo view "$REPO"` existe: **verifica que sea privado**
  (`gh repo view "$REPO" --json isPrivate -q .isPrivate` == `true`). Si es
  **público, ABORTA** y dilo (no subas config a un repo público; ofrece
  `gh repo edit "$REPO" --visibility private` como paso que el usuario confirma).
- Si no existe: `gh repo create "$REPO" --private --description "Config de Claude Code (privado)"`.

## 6. Commit y push
- `git init` en el área de ensayo (o clona el repo si ya tiene historia y copia
  encima), añade un `.gitignore` con las exclusiones del paso 3, `git add -A`.
- Si no hay cambios, dilo y termina.
- Commit (`Respaldo de config de Claude Code <fecha>`), `git push` a `$REPO`
  (rama `main`).

## 7. Reporte
Di: qué rutas se respaldaron, el resultado del escáner de secretos (**debe decir
"0 hallazgos"**), el repo (`$REPO`, privado) y el SHA. Nunca imprimas un secreto.
