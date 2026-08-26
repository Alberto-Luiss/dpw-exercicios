# E00.4 — Desfazer sem pânico

## Tabela

| # | Cenário | Comando |
|---|---|---|
| 1 | Editei um arquivo e quero descartar a alteração (ainda não fiz `add`) | `git restore <arquivo>` |
| 2 | Fiz `git add` do arquivo errado e quero tirá-lo do stage | `git restore --staged <arquivo>` |
| 3 | A mensagem do último commit está errada (ainda não fiz push) | `git commit --amend -m "nova mensagem"` |
| 4 | Quero desfazer o último commit, mas manter as alterações no working directory | `git reset --soft HEAD~1` |
| 5 | Quero reverter um commit já enviado para o remoto | `git revert <hash>` |

## Caso 1 — Restaurar alteração não commitada

Antes:
```
On branch main
Your branch is up to date with 'origin/main'.
Changes not staged for commit:
        modified:   README.md
no changes added to commit (use "git add" and/or "git commit -a")
```

```powershell
git restore README.md
```

Depois:
```
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

## Caso 2 — Tirar do stage

Staged:
```
On branch main
Changes to be committed:
        modified:   README.md
```

```powershell
git restore --staged README.md
```

Unstaged (edição continua no arquivo):
```
On branch main
Changes not staged for commit:
        modified:   README.md
no changes added to commit (use "git add" and/or "git commit -a")
```

## Caso 3 — Corrigir mensagem do último commit

```powershell
git commit -m "escrevi errado"
git log --oneline -3
```
```
f6d212a (HEAD -> main) escrevi errado
a916343 (origin/main) docs: evidencia do exercicio 3
d2c6dbb Merge branch 'feat/titulo-b' into main
```

```powershell
git commit --amend -m "docs: corrigindo erro no commit"
git log --oneline -3
```
```
246827c (HEAD -> main) docs: corrigindo erro no commit
a916343 (origin/main) docs: evidencia do exercicio 3
d2c6dbb Merge branch 'feat/titulo-b' into main
```

O hash mudou de `f6d212a` para `246827c` — o `--amend` reescreve o commit, não empilha um novo.

## Caso 4 — Desfazer commit mantendo alterações

Antes:
```
246827c (HEAD -> main) docs: corrigindo erro no commit
a916343 (origin/main) docs: evidencia do exercicio 3
d2c6dbb Merge branch 'feat/titulo-b' into main
```
```
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
nothing to commit, working tree clean
```

```powershell
git reset --soft HEAD~1
```

Depois:
```
a916343 (HEAD -> main, origin/main) docs: evidencia do exercicio 3
d2c6dbb Merge branch 'feat/titulo-b' into main
c7ac63d (feat/titulo-b) docs: readme inicial - versao B
```
```
On branch main
Your branch is up to date with 'origin/main'.
Changes to be committed:
        modified:   README.md
```

O commit `246827c` sumiu do histórico, mas a edição do `README.md` voltou como staged — nada se perdeu, só o commit foi desfeito.

## Caso 5 — Reverter commit já enviado ao remoto

```powershell
git revert a916343
```
```
[main c75d3a3] Revert "docs: evidencia do exercicio 3"
 1 file changed, 55 deletions(-)
 delete mode 100644 evidencias/e3-conflito.md
```

Link do commit de revert: https://github.com/Alberto-Luiss/dpw-exercicios/commit/c75d3a3

## Reflog final

```powershell
git reflog -10
```
```
7359622 (HEAD -> main, origin/main, origin/HEAD) HEAD@{0}: pull: Merge made by the 'ort' strategy.
c75d3a3 HEAD@{1}: revert: Revert "docs: evidencia do exercicio 3"
a916343 HEAD@{2}: reset: moving to HEAD~1
246827c HEAD@{3}: commit (amend): docs: corrigindo erro no commit
f6d212a HEAD@{4}: commit: escrevi errado
a916343 HEAD@{5}: commit: docs: evidencia do exercicio 3
d2c6dbb HEAD@{6}: commit (merge): Merge branch 'feat/titulo-b' into main
5b27ac0 (feat/titulo-a) HEAD@{7}: checkout: moving from feat/titulo-b to main
c7ac63d (feat/titulo-b) HEAD@{8}: commit: docs: readme inicial - versao B
bb02f76 HEAD@{9}: checkout: moving from main to feat/titulo-b
```

Repare que só os casos 3, 4 e 5 aparecem no reflog (`commit (amend)`, `reset`, `revert`) — os casos 1 e 2 mexeram no working directory e no stage, não no histórico, então não deixam rastro ali.

## Por que o caso 5 é diferente do caso 4?

No caso 4, o commit desfeito ainda estava só na minha máquina, reescrever o histórico local não afeta ninguém. No caso 5, o commit já tinha sido enviado ao remoto, então reescrever o histórico (com `reset` ou `amend`) quebraria o repositório de qualquer pessoa que já tivesse baixado esse commit. O `revert` resolve isso criando um commit novo que desfaz as mudanças, sem apagar nada que já existe.