# E00.5 — Roteiro de diagnóstico

Cenário: "Instalei o pacote, mas o `import` fala que não existe."

## Roteiro (do mais barato ao mais caro)

| # | Comando | Se a saída for X | Então |
|---|---|---|---|
| 1 | `Select-String "nome-do-pacote" package.json` | Não aparece nada | O pacote nunca foi salvo como dependência — rodar `pnpm add nome-do-pacote` |
| 2 | `Test-Path node_modules\nome-do-pacote` | Retorna `False` | Declarado no package.json mas não instalado fisicamente — rodar `pnpm install` |
| 3 | Comparar o nome usado no `import` com a pasta real em `node_modules\` | Nomes diferentes (digitação, capitalização, ou faltou `@escopo/`) | Corrigir o nome no código |
| 4 | `git branch --show-current` e `pwd` | Branch ou pasta diferente da esperada | Ambiente instalado não corresponde ao código atual — trocar branch/pasta e rodar `pnpm install` de novo |
| 5 | `Remove-Item -Recurse -Force node_modules; pnpm install` | Ainda falha depois disso | Cache/manifesto do pnpm corrompido, registry fora do ar, ou o pacote realmente não existe no npm |

Cada passo elimina uma hipótese específica antes de avançar para o próximo, do mais barato
(ler um arquivo) para o mais caro (reinstalar tudo do zero).

## Demonstração com falha provocada

Instalei um pacote de teste e confirmei que funcionava:

```powershell
pnpm add lodash
node teste-diagnostico.js
```
```
Funcionando
```

Quebrei o ambiente de propósito, apagando o pacote de dentro de `node_modules` sem
desinstalar pelo pnpm (simulando uma instalação corrompida):

```powershell
Remove-Item -Recurse -Force node_modules\.pnpm\lodash*
Remove-Item -Recurse -Force node_modules\lodash
node teste-diagnostico.js
```
```
Error [ERR_MODULE_NOT_FOUND]: Cannot find package 'lodash' imported from
C:\dev\dpw-exercicios\teste-diagnostico.js
```

Apliquei o roteiro:

**Passo 1** — o pacote está no `package.json`?
```powershell
Select-String "lodash" package.json
```
```
package.json:27:    "lodash": "^4.18.1"
```
Está declarado — hipótese 1 descartada.

**Passo 2** — está fisicamente instalado?
```powershell
Test-Path node_modules\lodash
```
```
False
```
Causa encontrada: declarado mas ausente fisicamente.

**Tentativa de correção simples:**
```powershell
pnpm install
```
```
Already up to date
Done in 49ms using pnpm v11.23.0
```
O erro persistiu. O pnpm mantém um manifesto interno de que "já instalou" tudo, e como a
pasta foi apagada manualmente (por fora do pnpm), ele não detectou a inconsistência.

**Passo 5 — reinstalação completa:**
```powershell
Remove-Item -Recurse -Force node_modules
pnpm install
node teste-diagnostico.js
```
```
Packages: +2
dependencies:
+ lodash 4.18.1
devDependencies:
+ prettier 3.9.6
Done in 930ms using pnpm v11.23.0

Funcionando
```

Causa raiz confirmada: instalação fisicamente incompleta que o `pnpm install` simples não
detecta sozinho, exigindo remoção total de `node_modules` para forçar a reconstrução.