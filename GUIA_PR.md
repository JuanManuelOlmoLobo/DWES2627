# Guía genérica: cómo hacer una Pull Request (PR)

Guía aplicable a cualquier repositorio de GitHub. Cambia los valores en MAYÚSCULAS por los tuyos:

- `PROPIETARIO/REPO` → repositorio original (ej. `mamagar403-beep/DWES2627`)
- `TU-USUARIO` → tu usuario de GitHub
- `mi-rama` → nombre descriptivo de tu rama

---

## 0. Requisitos

- Cuenta de GitHub.
- Git instalado (`git --version`).
- Opcional: GitHub CLI (`gh --version`) para crear la PR desde la terminal.

Configura tu identidad una sola vez:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

---

## 1. Elige tu flujo de trabajo

### Flujo A — Tienes permiso de escritura en el repo
No necesitas fork. Clona el repo, crea una rama y sube esa rama.

```bash
git clone https://github.com/PROPIETARIO/REPO.git
cd REPO
git checkout -b mi-rama
```

### Flujo B — No tienes permiso (fork)
Haz **Fork** en GitHub (botón arriba a la derecha). Después:

```bash
git clone https://github.com/TU-USUARIO/REPO.git
cd REPO
git remote add upstream https://github.com/PROPIETARIO/REPO.git
git checkout -b mi-rama
```

Comprueba los remotos:

```bash
git remote -v
# origin   → tu fork (donde subes)
# upstream → repo original (del que te actualizas)
```

---

## 2. Mantén tu `main` actualizado (flujo B)

Antes de empezar, sincroniza con el original:

```bash
git checkout main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
```

Y crea la rama de trabajo desde ahí:

```bash
git checkout -b mi-rama
```

> Regla de oro: **una rama por cambio**. Nunca trabajes directamente en `main`.

---

## 3. Haz el cambio y haz commit

```bash
# edita los archivos
git add .
git commit -m "tipo: descripción corta del cambio"
```

### Mensajes de commit recomendados (Conventional Commits)

| Tipo | Uso |
|---|---|
| `feat` | nueva funcionalidad |
| `fix` | corrección de un error |
| `docs` | documentación |
| `refactor` | reestructuración sin cambiar comportamiento |
| `test` | pruebas |
| `chore` | tareas de mantenimiento |

Ejemplo:

```bash
git commit -m "docs: añadir guía de contribución"
```

Consejos:

- Commits pequeños y con un único propósito.
- Mensaje en imperativo y en minúsculas tras el tipo.
- No incluyas archivos generados, secretos ni binarios innecesarios.

Puedes revisar lo que vas a subir con:

```bash
git status
git diff
git log --oneline -5
```

---

## 4. Sube la rama

```bash
git push -u origin mi-rama
```

`-u` enlaza la rama local con la remota; los siguientes `push` basta con `git push`.

Git imprimirá un enlace para crear la PR, por ejemplo:

```
https://github.com/PROPIETARIO/REPO/pull/new/mi-rama
```

---

## 5. Crea la Pull Request

### Opción web (la más simple)

1. Abre el enlace que imprime `git push` (o ve al repo original → pestaña **Pull requests** → **New pull request**).
2. Verifica la comparación:
   - **base**: `PROPIETARIO/REPO` → `main`
   - **compare**: `TU-USUARIO/REPO` → `mi-rama`
3. Revisa que solo aparezcan los commits y archivos esperados.
4. Escribe un título claro y una descripción con:
   - **Qué** cambia.
   - **Por qué**.
   - **Cómo probarlo** (si aplica).
   - Issues relacionados: `Closes #12`.
5. Pulsa **Create pull request**.

### Opción CLI

```bash
gh auth login   # una sola vez
gh pr create \
  --repo PROPIETARIO/REPO \
  --base main \
  --head TU-USUARIO:mi-rama \
  --title "docs: descripción corta" \
  --body "Qué cambia y por qué"
```

Si tienes permiso de escritura y trabajas sobre el propio repo, basta con:

```bash
gh pr create --fill
```

---

## 6. Durante la revisión

- Atiende los comentarios y haz los cambios en la **misma rama**:

  ```bash
  git add .
  git commit -m "fix: aplicar comentarios de revisión"
  git push
  ```

  La PR se actualiza automáticamente.

- Si la rama base avanzó y hay conflictos o la PR está desactualizada:

  ```bash
  git fetch upstream
  git checkout mi-rama
  git merge upstream/main
  # resuelve conflictos, luego:
  git add .
  git commit
  git push
  ```

  (También vale `git rebase upstream/main` si el repo lo prefiere.)

- No abras una PR nueva para corregir la misma PR.

---

## 7. Después de mergear

```bash
git checkout main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
git branch -d mi-rama
git push origin --delete mi-rama
```

---

## 8. Buenas prácticas

- Una PR = un cambio lógico. Evita mezclar refactors con funcionalidades.
- PRs pequeñas se revisan antes y mejor.
- Describe el cambio; no dejes la descripción vacía.
- No incluyas credenciales, tokens ni archivos pesados innecesarios.
- Comprueba que el proyecto compila / los tests pasan antes de abrir la PR.
- Respeta el estilo y las convenciones del repositorio (mira sus `CONTRIBUTING.md`).
- Sé respetuoso en la revisión: critica el código, no a la persona.

---

## 9. Solución de problemas

| Problema | Causa habitual | Solución |
|---|---|---|
| `git push` falla con `HTTP 400` | límite/problema del transporte HTTP | `git -c http.version=HTTP/1.1 push` |
| `Permission denied` al hacer push | credenciales caducadas o repo ajeno | Reautentica o usa tu fork |
| `gh` dice `missing required scope 'read:org'` | token sin ese permiso | `gh auth login` con scope `read:org` |
| `Updates were rejected` | la rama remota tiene commits que no tienes | `git pull --rebase` y vuelve a subir |
| La PR muestra archivos que no querías | rama creada desde una base equivocada | Crea una rama limpia desde `upstream/main` |
| Conflictos al mergear | la base cambió | Actualiza tu rama y resuelve conflictos |
| No aparece el botón de PR | la rama no está subida | `git push -u origin mi-rama` |

---

## Resumen mínimo

```bash
# 1. fork en GitHub (si no tienes permisos)
git clone https://github.com/TU-USUARIO/REPO.git && cd REPO
git remote add upstream https://github.com/PROPIETARIO/REPO.git

# 2. rama
git checkout -b mi-rama

# 3. cambio + commit
git add . && git commit -m "docs: mi cambio"

# 4. subir
git push -u origin mi-rama

# 5. abrir la PR con el enlace que imprime git (o con `gh pr create`)
```
