# Agentic Workspace

Un workspace **agnóstico al proyecto**: un directorio paraguas, con su propio repo,
que aloja tu harness de agentes (skills, reglas, flujos) desacoplado de cualquier
proyecto. Traés el repo que quieras adentro y lo trabajás con TU harness — no al revés.

## Idea central

El harness es el que manda. Los proyectos son material de trabajo intercambiable que
entra y sale. Esto es inversión de dependencias: el proyecto depende del workspace,
nunca el workspace de un proyecto.

## Cómo se usa (flujo previsto)

1. Abrís Claude Code parado en la **raíz** de este workspace → tu harness carga.
2. `sync-repo <url|nombre>` → clona el proyecto en `repos/<nombre>/`.
3. Trabajás. Si necesitás paralelizar → `spawn-worktree <repo> <task>` → agente aislado
   en `.worktrees/<repo>/<task>/`.
4. Al terminar → merge, limpieza (`git worktree prune`), el worktree desaparece.

## Estado

- **Etapa 1 (actual):** cimientos — estructura, reglas, `.gitignore`, diseño escrito.
- **Próximas etapas:** skills `sync-repo` y `spawn-worktree`, y el resto del harness.

Ver el diseño completo en `docs/superpowers/specs/`.
