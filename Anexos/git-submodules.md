# Git Submodules
Los **git submodules** permiten incluir un repositorio de Git dentro de otro repositorio. Son ideales para gestionar dependencias externas, librerías compartidas o cualquier componente que viva en su propio repositorio pero que necesites integrar en un proyecto más grande.

- [Git Submodules](#git-submodules)
    - [¿Cómo funciona internamente?](#cómo-funciona-internamente)
    - [Comandos Básicos](#comandos-básicos)
    - [Actualizar el Submódulo desde el Repositorio Principal](#actualizar-el-submódulo-desde-el-repositorio-principal)
    - [Trabajar Directamente en el Repositorio del Submódulo](#trabajar-directamente-en-el-repositorio-del-submódulo)
    - [Sincronizar el Repositorio Principal después de un Pull](#sincronizar-el-repositorio-principal-después-de-un-pull)
    - [Ejecutar Comandos en Todos los Submódulos](#ejecutar-comandos-en-todos-los-submódulos)
    - [Flujo de Trabajo Recomendado](#flujo-de-trabajo-recomendado)
    - [Resumen Rápido de Comandos](#resumen-rápido-de-comandos)


## ¿Cómo funciona internamente?

Cuando agregas un submódulo, Git registra dos cosas:

- **`.gitmodules`** — archivo en la raíz del repositorio principal que lista los submódulos (URL y ruta).
- **Un commit pointer** — el repositorio principal guarda un *hash* exacto del commit al que apunta el submódulo, no el código en sí.

Esto significa que el repositorio principal **no versiona el contenido** del submódulo, solo un puntero a un commit específico.

---

## Comandos Básicos

### Agregar un submódulo

```bash
git submodule add <url-del-repositorio> [ruta/destino]
```

**Ejemplo:**
```bash
git submodule add https://github.com/usuario/libreria.git libs/libreria
```

Esto crea la carpeta `libs/libreria`, clona el repo ahí y genera/actualiza el archivo `.gitmodules`. Después debes hacer commit:

```bash
git add .gitmodules libs/libreria
git commit -m "feat: agrega submódulo libreria"
```

---

### Clonar un repositorio que tiene submódulos

Al clonar un repo con submódulos, las carpetas aparecen **vacías** por defecto. Tienes dos opciones:

**Opción A — Clone directo con submódulos:**
```bash
git clone --recurse-submodules <url-del-repositorio>
```

**Opción B — Ya clonaste el repo y las carpetas están vacías:**
```bash
git submodule init       # Registra los submódulos en .git/config
git submodule update     # Descarga el contenido de cada submódulo
```

O en un solo comando:
```bash
git submodule update --init --recursive
```

> `--recursive` es útil cuando los submódulos a su vez tienen sus propios submódulos.

---

### Ver el estado de los submódulos

```bash
git submodule status
```

El prefijo indica el estado:
- ` ` (espacio) — el submódulo está en el commit correcto
- `-` — el submódulo no ha sido inicializado
- `+` — el submódulo apunta a un commit diferente al registrado
- `U` — hay conflictos de merge

---

### Listar los submódulos configurados

```bash
cat .gitmodules
```

---

### Eliminar un submódulo

No hay un comando único, se hace en varios pasos:

```bash
# 1. Remover la entrada del índice y la carpeta
git rm <ruta/del/submodulo>

# 2. Eliminar la sección en .git/config
git submodule deinit -f <ruta/del/submodulo>

# 3. Borrar la carpeta interna de Git
rm -rf .git/modules/<ruta/del/submodulo>

# 4. Hacer commit
git commit -m "chore: elimina submódulo <nombre>"
```

---

## Actualizar el Submódulo desde el Repositorio Principal

### Traer los últimos cambios del submódulo (tracking remoto)

```bash
# Moverse al directorio del submódulo y hacer pull
cd libs/libreria
git pull origin main

# Regresar al repo principal y registrar el nuevo commit
cd ../..
git add libs/libreria
git commit -m "chore: actualiza submódulo libreria al último commit"
```

O directamente desde la raíz del repo principal:

```bash
git submodule update --remote libs/libreria
```

> `--remote` hace que Git use el branch configurado del remoto en lugar del commit registrado.

### Configurar el branch que sigue el submódulo

Por defecto, `--remote` usa `main`. Para cambiarlo:

```bash
git config -f .gitmodules submodule.libs/libreria.branch develop
```

Luego confirmas el cambio:
```bash
git add .gitmodules
git commit -m "chore: submódulo sigue branch develop"
```

### Actualizar TODOS los submódulos de golpe

```bash
git submodule update --remote --merge
```

- `--merge` integra los cambios remotos con tu trabajo local en el submódulo.
- Puedes usar `--rebase` en lugar de `--merge` según tu flujo de trabajo.

---

## Trabajar Directamente en el Repositorio del Submódulo

El submódulo es un repositorio Git completo. Puedes trabajar en él de forma independiente:

```bash
# Entrar al submódulo
cd libs/libreria

# El submódulo puede estar en "detached HEAD", hay que hacer checkout a un branch
git checkout main

# Hacer cambios, commits y push normalmente
git add .
git commit -m "fix: corrige bug en la función X"
git push origin main

# Regresar al repo principal y registrar el nuevo puntero
cd ../..
git add libs/libreria
git commit -m "chore: actualiza submódulo con fix de función X"
git push
```

> ⚠️ **Importante:** Si olvidas hacer `git checkout <branch>` dentro del submódulo, estarás en modo *detached HEAD* y tus commits podrían perderse.

---

## Sincronizar el Repositorio Principal después de un Pull

Cuando alguien más actualiza el puntero del submódulo y haces `git pull` en el repo principal, las carpetas del submódulo no se actualizan automáticamente. Debes ejecutar:

```bash
git pull
git submodule update --init --recursive
```

Para automatizar esto puedes configurar Git:

```bash
git config --global submodule.recurse true
```

Con esta opción, `git pull` actualiza los submódulos automáticamente.

---

## Ejecutar Comandos en Todos los Submódulos

```bash
git submodule foreach '<comando>'
```

**Ejemplos prácticos:**

```bash
# Ver el branch actual en cada submódulo
git submodule foreach 'git branch'

# Hacer pull en todos los submódulos
git submodule foreach 'git pull origin main'

# Ver el status de cada submódulo
git submodule foreach 'git status'
```

---

## Flujo de Trabajo Recomendado

```
┌─────────────────────────────────────────────────────────┐
│                   REPOSITORIO PRINCIPAL                 │
│                                                         │
│  1. git submodule update --init --recursive             │
│     (al clonar o hacer pull)                            │
│                                                         │
│  2. cd libs/libreria                                    │
│     git checkout main        ← salir de detached HEAD  │
│     (hacer cambios...)                                  │
│     git commit -m "..."                                 │
│     git push origin main                                │
│                                                         │
│  3. cd ../..                                            │
│     git add libs/libreria                               │
│     git commit -m "chore: actualiza submódulo"          │
│     git push                                            │
└─────────────────────────────────────────────────────────┘
```

---

## Resumen Rápido de Comandos

| Acción | Comando |
|---|---|
| Agregar submódulo | `git submodule add <url> [ruta]` |
| Clonar con submódulos | `git clone --recurse-submodules <url>` |
| Inicializar submódulos | `git submodule update --init --recursive` |
| Ver estado | `git submodule status` |
| Actualizar al último commit remoto | `git submodule update --remote` |
| Ejecutar comando en todos | `git submodule foreach '<cmd>'` |
| Eliminar submódulo | `git rm <ruta>` + `git submodule deinit` |
| Auto-actualizar con pull | `git config --global submodule.recurse true` |

---

> 💡 **Tip final:** Si los submódulos te generan demasiada complejidad, considera alternativas como [git subtree](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging#_subtree_merge) o gestores de paquetes (npm, pip, etc.) según el tipo de proyecto.
