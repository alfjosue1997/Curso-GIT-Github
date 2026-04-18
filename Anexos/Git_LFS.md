# Archivos Largos: Git Large File Storage `git-LFS`

# Indice

- [Archivos Largos: Git Large File Storage `git-LFS`](#archivos-largos-git-large-file-storage-git-lfs)
- [Indice](#indice)
- [Instalación](#instalación)
  - [Git-LFS para Windows](#git-lfs-para-windows)
  - [Git-LFS para LINUX](#git-lfs-para-linux)
- [`find`: Buscar archivos Grandes](#find-buscar-archivos-grandes)
- [`.gitattributes`](#gitattributes)
- [Uso de Git LFS](#uso-de-git-lfs)
- [FAQ](#faq)
  - [Añadir archivos que ya estaban en el historial de Git](#añadir-archivos-que-ya-estaban-en-el-historial-de-git)
  - [Deseo unificar el seguimiento de un archivo a un patron](#deseo-unificar-el-seguimiento-de-un-archivo-a-un-patron)
- [Desinstalar](#desinstalar)
- [Limitaciones](#limitaciones)


[Git-LFS](https://github.com/git-lfs/git-lfs) es una extensión en línea de comandos para manejar grandes archivos con Git.

**GitHub** solo puede registrar y subir archivos de cierto tamaño por lo que el máximo tamaño que se permitira para GitHub sera de **10MB**. Cualquier archivo más grande deberá ser registrado en Git utilizando **LFS** para poder subirlo a GitHub sin problemas.

# Instalación

## [Git-LFS para Windows](https://git-lfs.com/)

## [Git-LFS para LINUX](https://github.com/git-lfs/git-lfs/blob/main/INSTALLING.md)

# `find`: Buscar archivos Grandes

Para buscar archivos grandes en Linux se puede usar el siguiente comando en la terminal para buscar archivos de mas de 10 MegaBytes:

```bash
find . -type f -size +10M 
```

Donde:

- `find`: programa que busca archivos/dirs según criterios.
- `.` : directorio donde iniciar la búsqueda (puedes reemplazar por `/ruta`).
- `-type f`: restringe a archivos regulares (no directorios, enlaces, etc.).
- `-size +10M`: tamaño del archivo; `10M` = 10 MegaBytes. El prefijo:
  - `+10M`  = mayor que 10MB
  - `-10M`  = menor que 10MB
  - `10M`   = exactamente 10MB
  
De manera general, el comando es:

```bash
find [ruta] [opciones] [expresión]
```

# `.gitattributes`

Para el funcionamiento de `git lfs` es necesario crear el siguiente archivo, en Linux:

```bash
touch .gitattributes
```

`git lfs` modificará este archivo y almacenará como git debe trackear aquellos archivos grandes que nosotros escojamos.

# Uso de Git LFS

> **NOTA**: Se recomienda que al empezar un proyecto que este destinado a tener grandes archivos se comience revisando el tamaño de los archivos usando `find` he inmediatamente añadir los más pesados al staging area para evitar dificultades.

Para trackear un archivo, es decir lo mismo que harías con `git add`, simplemente podemos usar:

```bash
git lfs track "<ruta/al/archivo.ext>"
```

> Nota: Las comillas `""` son importantes. \
> *Previene que quien interprete la ruta sea el shell*.

Esto editará el archivo `.gitattributes` y agrega la siguiente línea:

```bash
<ruta/al/archivo.ext> filter=lfs diff=lfs merge=lfs -text
```

Para agrupar varios archivos del mismo tipo se puede usar:

```bash
git lfs track "*.ext"
```

**Git LFS** es apropiado cuando trabajas con **archivos grandes**, **binarios** o **no diffs** que:

- Cambian con frecuencia
- No pueden fusionarse (merge) de forma semántica
- Inflan el historial del repositorio Git
- No aportan valor al versionado línea por línea

Despues de haber usado `git lfs track` debes hacer commit en los cambios hechos al archivo `.gitattributes`:

```bash
git add .gitattributes
git commit -m "Git LFS track para archivos.ext"
```

Ahora puedes usar el repositorio de Git Como usualmente lo harías. **Git LFS** se encargará de gestionar los archivos de manera automática. 

```bash
git add archivo.ext
git commit -m "añadido archivo.ext"
```

Puedes confirmar que Git LFS esta manejando tus archivos:

```bash
git lfs ls-files
```

Una vez hecho commit, has push a tus archivos a GitHub:

```bash
git push origin main
```

# FAQ

## Añadir archivos que ya estaban en el historial de Git

`git lfs track` No trackea de manera retroactiva, solo edita el archivo `.gitattributes` para que luego `git` pueda guardar el historial de otra forma, más ligera.

El flujo de trabajo para poder trackear archivos grandes que ya estaban en el historial depende sí el tamaño del historial es importante y si esta compartido o no con otros colaboradores. Podemos dividirlo en los siguientes casos:

### Sin modificar el historial de los archivos

Cuando el proyecto esta publicado en Github, o si el repo ya es compartido entonces para no romper forks ni PRs basta con que Git LFS se aplique a partir de ahora en el nuevo seguimiento. 

1. Añade el archivo a Git LFS. Ya sea el archivo individual:

    ```bash
    git lfs track "<ruta/al/archivo.bin>"   
    ```

    O el patron:

    ```bash
    git lfs track "*.bin"    
    ```

    Esto añade a `.gitattributes`:

    ```bash
    <ruta/al/archivo.bin> filter=lfs diff=lfs merge=lfs -text
    *.bin filter=lfs diff=lfs merge=lfs -text
    ```

2. Re-agregar los `.bin` para que **Git LFS** los gestione correctamente, ya sea el archivo inididual:

    ```bash
    git rm --cached <ruta/al/archivo.bin>
    git add archivo.bin
    ```

    O por patron:

    ```bash
    git rm --cached "*.bin"
    git add archivo.bin
    ```

    `git rm --cached` solo elimina el archivo  del índice (staging). Esto es necesario porque:

    - Git ya había registrado el archivo.
    - Cambiar `.gitattributes` no aplica retroactivamente.
    - Debes forzar que Git lo vuelva a indexar como LFS.

    El archivo sigue existiendo en commits anteriores y en el working directory. En el commit siguiente el archivo se vuelve a añadir, ahora bajo la nueva regla de LFS

    > Cabe aclarar que `git rm --cached` **NO** borra el historial ni lo reescribe.

3. Commit claro

    ```bash
    git commit -m "Reindex BIN files under Git LFS"
    ```

### Modificando el historial de los archivos

Cuando el proyecto **no** esta publicado en Github, o si el repo **no** es compartido o si los colaboradores pueden coordinarse para re-clonar el proyecto, entonces podemos editar el historial completo para que sea mas limpio. Para ello se utilizará [`git lfs migrate`](https://github.com/git-lfs/git-lfs/blob/main/docs/man/git-lfs-migrate.adoc).

```bash
# Un solo archivo
git lfs migrate import --include="<ruta/al/archivo.bin>"
# Por Patron
git lfs migrate import --include="*.bin"
```

Con esto:

- Todos los commits anteriores se modifican
- Los `.bin` pasan a LFS **en toda la historia**
- Cambian los hashes de commit

Puedes verificar el `.gitattributes`, debe contener:

```bash
# Un solo archivo
<ruta/al/archivo.bin> filter=lfs diff=lfs merge=lfs -text
# Por Patron
*.bin filter=lfs diff=lfs merge=lfs -text
```

Comunicación obligatoria al equipo. Debes avisar explícitamente que el historial fue reescrito debido a migración a Git LFS y que todos deben eliminar clones antiguos y volver a clonar.

Se recomienda usar `git lfs migrate` sí:

- El repo es nuevo o controlado
- El tamaño del repo ya es un problema serio
- Todos los colaboradores pueden re-clonar
- Estás dispuesto a forzar pushes

## Deseo unificar el seguimiento de un archivo a un patron

### 1. Verifica cómo está el archivo actualmente en `.gitattributes`

Ejemplo actual (mala escalabilidad):

```bash
$ cat .gitattributes
archivo.bin filter=lfs diff=lfs merge=lfs -text
```

### 2. Añade el patrón general

```bash
git lfs track "*.bin"    
```

Esto añade a `.gitattributes` la ultima línea:

```bash
$ cat .gitattributes
archivo.bin filter=lfs diff=lfs merge=lfs -text
*.bin filter=lfs diff=lfs merge=lfs -text
```

### 3. (*Recomendado*) Elimina la regla individual redundante. Edita `.gitattributes` y deja solo:

```bash
*.bin filter=lfs diff=lfs merge=lfs -text
```

Así el codigo queda más limpio y sin duplicaciones. Por ultivo re-agregar los `.bin` para que **Git LFS** los gestione correctamente

```bash
git rm --cached archivo.bin
git add archivo.bin
```

`git rm --cached` solo elimina el archivo  del índice (staging). Esto es necesario porque:

- Git ya había registrado el archivo.
- Cambiar `.gitattributes` no aplica retroactivamente.
- Debes forzar que Git lo vuelva a indexar como LFS.

El archivo sigue existiendo en commits anteriores y en el working directory. En el commit siguiente el archivo se vuelve a añadir, ahora bajo la nueva regla de LFS

> Cabe aclarar que `git rm --cached` **NO** borra el historial ni lo reescribe.

#### 4. Commit claro

```bash
git commit -m "Unify Git LFS tracking for BIN files"
```

# Desinstalar

Se puede regresar el repositorio a un repo normal de Git repository con `git lfs migrate`. Por ejemplo:

```bash
git lfs migrate export --include="*.bin"
```

>**Nota:** Esta acción reescribe el historial y cambia todos los Git object ID en el repositorio. Igual que la versión `import` de este comando.

# Limitaciones
Git LFS mantiene una [lista](https://github.com/git-lfs/git-lfs/wiki/Limitations) de limitaciones conocidas que puedes buscar y editar si quieres.

Git LFS trabajará con versiones de Git tan tempranas como Git 2.0.0. Sin embargo, para mejor rendimiento se recomienda usar una versión de Git reciente.
