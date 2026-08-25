# E00.1 — Ambiente reprodutível

## Reinstalação a partir do zero

```powershell
Remove-Item -Recurse -Force node_modules
pnpm install --frozen-lockfile
git status --short
```

Saída:

```
✓ Lockfile passes supply-chain policies (verified 32m ago)
Lockfile is up to date, resolution step is skipped
Packages: +1
+
Packages are hard linked from the content-addressable store to the virtual store.
  Content-addressable store is at: C:\Users\alber\AppData\Local\pnpm\store\v11
  Virtual store is at:             node_modules/.pnpm
Progress: resolved 1, reused 1, downloaded 0, added 1, done
devDependencies:
+ prettier 3.9.6
Done in 268ms using pnpm v11.23.0

(git status --short não retornou nada — repositório limpo)
```

`git status --short` retornou vazio, confirmando que a reinstalação não alterou nenhum
arquivo versionado.

## Link permanente do .gitignore

https://github.com/Alberto-Luiss/dpw-exercicios/blob/1c77a6cfe34b01b5cba0cd342c0ea26b27fd25a5/.gitignore#L1

## Por que o pnpm-lock.yaml é versionado e o node_modules/ não?

O `pnpm-lock.yaml` registra a versão exata de cada dependência (e das dependências das
dependências), garantindo que qualquer pessoa que instale o projeto tenha exatamente o
mesmo ambiente. Já o `node_modules/` é apenas o resultado dessa instalação — pode ser
recriado a qualquer momento a partir do lockfile, é pesado, e versionar ele só infla o
repositório sem agregar informação nova.