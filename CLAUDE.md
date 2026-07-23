# Agentic Workspace

Workspace paraguas para trabajar cualquier repo bajo un harness propio, desacoplado
de todo proyecto. Los proyectos son intercambiables; el harness manda.

## Reglas del workspace

1. **Abrí Claude Code SIEMPRE desde la raíz de este workspace.**
   No entres a un repo y abras Claude ahí — *traés el repo al workspace*. Solo así
   el harness de `skills/` queda cargado y manda. Si abrís desde `repos/<repo>/`,
   Claude carga los skills de ese proyecto y NO el harness. No entrás al repo:
   traés el repo al workspace.

2. **Los repos entran por el skill `sync-repo`.**
   Se clonan en `repos/<nombre>/` (gitignored), cada uno con su propio `.git`.

3. **El paralelismo va por el skill `spawn-worktree`.**
   Crea worktrees efímeros en `.worktrees/<repo>/<task>/`. NUNCA adentro del repo
   (evita escaneo recursivo y que un agente edite la rama de otro).

4. **Limpieza de worktrees:** correr `git worktree prune` ANTES de borrar el repo
   fuente. Git guarda paths absolutos en el gitdir; si movés el workspace, usar
   `git worktree repair`.

## Estructura

```
agentic-workspace/
├─ CLAUDE.md                 ← este archivo (reglas del workspace)
├─ README.md
├─ .gitignore               ← ignora /repos y /.worktrees
├─ skills/                  ← EL HARNESS. Versionado. Siempre activo.
├─ repos/                   ← proyectos clonados (gitignored)
└─ .worktrees/<repo>/<task>/ ← worktrees efímeros (gitignored, on-demand)
```

## Qué versiona este repo

Solo el harness: `skills/`, `CLAUDE.md`, `README.md`, `.gitignore`, `docs/`.
El código de los proyectos (`repos/`) y los worktrees (`.worktrees/`) NO.
