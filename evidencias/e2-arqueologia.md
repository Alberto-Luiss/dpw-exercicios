# E00.2 — Arqueologia de histórico

Repositório investigado: [nestjs/nest](https://github.com/nestjs/nest)

## 1. Quantos commits o repositório tem?

```powershell
git rev-list --count HEAD
```

```
21659
```

## 2. Qual foi o primeiro commit, e em que data?

```powershell
git log --reverse --format="%H %ad" --date=short | Select-Object -First 1
```

```
f7c8d10fb20943bc7102c73d5ecbe49e6c0b5ea1 2017-01-08
```

## 3. Quem mais modificou packages/core/injector/injector.ts?

```powershell
git shortlog -sn -- packages/core/injector/injector.ts
```

```
90  Kamil Myśliwiec
```

## 4. O que mudou no último commit que tocou esse arquivo?

```powershell
git log -1 --format="%H %ad %s" --date=short -- packages/core/injector/injector.ts
git show 45485b5 -- packages/core/injector/injector.ts
```

```
commit 45485b54210e06a517c1ebf86b42b1ea99fc3fe2
Author: Kamil Myśliwiec <mail@kamilmysliwiec.com>
Date:   Tue Aug 25 12:48:22 2026 +0200
    fix(core): circular durable providers issue #17562

diff --git a/packages/core/injector/injector.ts b/packages/core/injector/injector.ts
index 32b9f2850..aad3a899c 100644
--- a/packages/core/injector/injector.ts
+++ b/packages/core/injector/injector.ts
@@ -581,7 +581,7 @@ export class Injector {
        * that eventual lazily created instance will be merged with the prototype
        * instantiated beforehand.
        */
-      instanceHost.donePromise &&
+      if (instanceHost.donePromise) {
         void instanceHost.donePromise
           .then(() =>
             this.loadProvider(instanceWrapper, moduleRef, resolutionContext),
@@ -589,6 +589,20 @@ export class Injector {
           .catch(err => {
             instanceWrapper.settlementSignal?.error(err);
           });
+      } else {
+        await this.loadProvider(
+          instanceWrapper,
+          instanceWrapper.host ?? moduleRef,
+          resolutionContext,
+        );
+      }
     }
```

Resumo: o código antigo só reagia se já existisse um `donePromise` herdado; se não existisse,
nada acontecia, causando o bug #17562 (referências circulares com providers duráveis não
resolvidas). O fix adiciona um `else` que carrega o provider imediatamente quando não há
`donePromise` herdado.

## 5. Quantos commits foram feitos nos últimos 90 dias?

```powershell
git log --since="90 days ago" --oneline | Measure-Object -Line
```

```
694
```