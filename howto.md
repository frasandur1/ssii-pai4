# Cómo va el trasto este

La rama objetivo de nuestros cambios es siempre `develop`. Por tanto, crea tu
rama de trabajo desde ella:

```Shell
git switch develop
git pull
git switch -c feature/my-change
```

Haz el push normalmente. Al crear la PR, selecciona `develop` como rama
objetivo. Desde la CLI:

```Shell
git push --set-upstream origin feature/my-change

gh pr create `
  --base develop `
  --head feature/my-change
```

El workflow se ejecuta al abrir o reabrir una PR, al subir nuevos commits y al
marcar como lista para revisión una PR que estaba en borrador.

También se puede lanzar manualmente desde GitHub con:

```Shell
gh workflow run devsecops.yml --ref develop
# Después, usa el identificador que devuelve GitHub:
gh run watch <run-id>
```

