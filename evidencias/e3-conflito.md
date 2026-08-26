# E00.3 — Conflito de merge

## Saída do merge que acusou o conflito

```powershell
git switch main
git merge feat/titulo-b
```

```
Auto-merging README.md
CONFLICT (add/add): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

## Conteúdo do arquivo durante o conflito (antes de resolver)

```
<<<<<<< HEAD
# DPW — Exercícios do M00 (versão A)
=======
# DPW — Exercícios do M00 (versão B)
>>>>>>> feat/titulo-b
```

## Grafo do histórico

```powershell
git log --graph --oneline --all
```

```
*   d2c6dbb (HEAD -> main) Merge branch 'feat/titulo-b' into main
|\
| * c7ac63d (feat/titulo-b) docs: readme inicial - versao B
* | 5b27ac0 (feat/titulo-a) docs: readme inicial - versao A
|/
* bb02f76 (origin/main) docs: evidencia do exercicio 2
* 4fa65de docs: evidencia do exercicio 1
* 6b652e7 docs: evidencia do exercicio 1
* f3bd314 docs: evidencia do exercicio 1
* 1c77a6c chore: Ambiente inicial reprodutivel
```

## Links permanentes

- Commit de merge: https://github.com/Alberto-Luiss/dpw-exercicios/commit/d2c6dbb
- Grafo de branches: https://github.com/Alberto-Luiss/dpw-exercicios/network

## Por que o Git não conseguiu resolver sozinho?

As duas branches criaram o mesmo arquivo (`README.md`) de forma independente, partindo de um
ponto onde ele não existia. Como as duas versões divergem já na primeira linha e o Git não
tem como saber qual delas deveria "vencer", ele marca a área conflitante com os marcadores
`<<<<<<<`, `=======` e `>>>>>>>` e devolve a decisão para o autor.