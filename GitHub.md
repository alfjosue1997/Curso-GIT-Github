
# INDICE

- [INDICE](#indice)
- [Conexión SSH](#conexión-ssh)
  - [Creando un directorio para SSH](#creando-un-directorio-para-ssh)
  - [Creación de la llave RSA](#creación-de-la-llave-rsa)
  - [Configurando el Host](#configurando-el-host)
  - [Conexión de más de un usuario en el mismo equipo (opcional)](#conexión-de-más-de-un-usuario-en-el-mismo-equipo-opcional)
  - [Cambio de permisos](#cambio-de-permisos)
  - [Agregar la llave pública a GitHub](#agregar-la-llave-pública-a-github)
  - [Inciar sesión con SSH](#inciar-sesión-con-ssh)
  - [Conectando más de un usuario por computadora (opcional)](#conectando-más-de-un-usuario-por-computadora-opcional)
- [Enlace entre repositorios Git y GitHub](#enlace-entre-repositorios-git-y-github)
  - [Clonar un repositorio remoto](#clonar-un-repositorio-remoto)
  - [Subir un repositorio remoto](#subir-un-repositorio-remoto)
  - [Ramas en Github](#ramas-en-github)
- [Colaboración entre multiples personas](#colaboración-entre-multiples-personas)
  - [WorkFlow](#workflow)
  - [Fork](#fork)
  - [Upstream](#upstream)



# Conexión SSH 

En este punto se recomienda instalar WSL si estás en Windows para ejecutar comandos de Ubuntu en terminal, aunque no es obligatorio. Por tanto, la gran parte de los pasos es la misma en Linux (distribuciones basadas en Debian como Ubuntu o Mint), y Windows con WSL. 


Si se usa Windows nativo los siguientes comandos se deberán ejecutar desde Powershell, y comúnmente la diferencia entre un comando en Linux y en Powershell será la terminación <b>`.exe`</b> de ejecutable (ejemplo, `ssh` en Linux/WSL, y `ssh.exe` en Powershell). En ambos SOs usaremos la terminal, ya sea la de Linux, el WSL, o el Powershell de Windows.



## Creando un directorio para SSH

Para conectar el repositorio local con un repositorio de GitHub, usaremos la conexión SSH, para lo cual necesitamos una **clave RSA**. 

### Directorio SSH Linux/WSL
Primero, en el directorio `home` de Linux o de WSL del usuario en tu computadora (`~` o `/home/you_user` en Linux, la misma ruta en WSL Windows) crea una nueva carpeta <b>~/.ssh</b>, y abre la terminal dentro de ella. Puedes hacerlo con los siguientes comandos:

```bash
#Cambiar al home de tu usuario
cd ~
#Crear directorio
mkdir .ssh
#Entrar al directorio
cd .ssh
```
Si vas a usar WSL, es importante usar la ruta `~` del WSL en vez de tu carpeta de usuario personal en Windows (comunmente `C:\Users\your_user`) pues sólo puedes gestionar permisos con comandos de Unix dentro de los directorios de WSL, no en los nativos de Windows. 


### Directorio SSH Windows 10
Si usás **Windows 10** nativamente debés ir a tu `home` de usuario (la ruta `~ = C:\Users\your_user`), y crear la carpeta <b>`.ssh`</b>, posteriormente entrar en ella con un `cd`:

```Powershell
#Entrar a .ssh en Windows
cd C:\Users\your_user\.ssh
```


## Creación de la llave RSA

Ejecutá el siguiente comando para generar una llave RSA:

```bash
ssh-keygen -N "passphrase"
```

O si estás en Windows, el comando es:

```Powershell
ssh-keygen.exe -N "passphrase"
```

El argumento adicional `-N "passphrase"` es opcional y por tanto se puede omitir, sirve para crear la llave usando una frase o palabra secreta (puedes usar la que quieras en vez de "passphrase") como semilla generadora, pero la llave puede crearse sin usar una palabra secreta. Por mayor seguridad, se recomienda usar la passphrase, de este modo, las futuras conexiones la pedirán siempre que se haga una consulta con GitHub, sirviendo como una contraseña.

Al ejecutar el comando, pedirá que se ingrese un nombre para el archivo de la clave, puedes poner el que sea, por ejemplo, `my_key1`, y dicha clave se guardará en la carpeta <b>.ssh</b> donde estás ejecutando el comando. Si vas a la carpeta, verás que se han generado dos archivos, `my_key1` (sin extensión, pero puede abrirse con cualquier editor de texto plano como Bloc de Notas), el archivo de clave privada, y que por tanto no debe compartirse; y `my_key1.pub`, el archivo con la clave pública de cifrado. También puedes comprobar esto listando los archivos del directorio actual (en Linux):

```bash
ls -a
```

Y en Windows basta con abrir la carpeta <b>.ssh</b> en vez de ejecutar `ls` para ver que se han generado 2 archivos de clave.

## Configurando el Host

Dentro de la carpeta <b>.ssh</b> (todavía) debes crear un archivo llamado `config`. Desde la terminal Linux o WSL esto se hace con:

```bash
touch config
```

O en Windows simplemente con click derecho y "Crear Nuevo Documento". Este archivo lo podemos abrir con Bloc de Notas (o equivalentes en Linux), o también con VS Code:

```bash
code .
```

y agregamos lo siguiente (en Linux):

```text
#Primer host
Host github-host1
  HostName github.com
  User git
  IdentityFile ~/.ssh/my_key1
```

Y en Windows, debemos cambiar la ruta del IdentityFile:


```text
#Primer host
Host github-host1
  HostName github.com
  User git
  IdentityFile C:\Users\your_user\.ssh\my_key1
```

En donde puedes cambiar "github-host1" por cualquier nombre que quieras, pero se recomienda uno alusivo al repositorio y usuario (tú) para distinguirlo de cualquier otra conexión a SSH o a GitHub por parte de otros usuarios del equipo. De igual modo, en la línea de `IdentityFile` debes cambiar `my_key1` por el nombre de archivo que le pusiste a tu clave privada, pero <b>siempre escribiendo la ruta completa (no la relativa) de la clave</b>. Además, si el nombre de tu carpeta de usuario tiene espacios o caracteres especiales, la ruta se debe poner entre comillas, por ejemplo, `"C:\Users\tu usuario\.ssh\my_key1"`.

## Conexión de más de un usuario en el mismo equipo (opcional)
Esta parte es opcional.

Si se requiriera conectar más usuarios ya sea a éste o a otros repositorios GitHub, se pueden copiar las líneas anteriores para cada usuario y añadirlas al archivo `config`, siempre y cuando los nombres del `Host` sean distintos para cada uno. Además, respeta la identación de cada Host de la lista. Un ejemplo de `config` para dos usuarios sería el siguiente:

```text
#Primer host (para el primer usuario)
Host github-host1
  HostName github.com
  User git
  IdentityFile ~/.ssh/my_key1

#Segundo host (para el segundo usuario)
Host github-host2
  HostName github.com
  User git
  IdentityFile ~/.ssh/my_key2
```

Y recordando que si estás en Windows, debes cambiar las rutas a las llaves privadas con una ruta de Windows, como `C:\Users\your_user\.ssh\my_key`.

## Cambio de permisos

Para que la conexión SSH funcione, se requiere que el archivo de clave privada sólo tenga permisos de acceso para el propietario del archivo. Normalmente el comando `ssh-keygen` asigna los permisos adecuados de forma automática al crear la llave (tanto en Windows como en Linux), pero en caso de que no ocurra, se pueden estableccer en cualquier momento. Para asegurar estos permisos en Linux y WSL, se debe ejecutar lo siguiente aún dentro del directorio `~/.ssh` (lo cual aseguras con un `cd ~/.ssh`):

```bash
chmod 600 my_key1
```

Y en Windows, basta con dirigirse a <b>.ssh</b>, dar click derecho al archivo de llave privada, y cambiar los permisos, asegurando que sólo el administrador pueda leer y escribir en él.

**RECOMENDACIÓN:** Abrir ventana de GIT bash y ejecutar la siguiente linea. 
```bash
chmod 600 my_key1
```

## Agregar la llave pública a GitHub
**[Agregar imagenes de esta sección]**
Ahora, debes abrir tu repositorio de GitHub, o uno perteneciente a alguna organización (ejemplo, LINX) una vez que el dueño de ese repositorio te haya dado los permisos para editarlo. 

Dentro del repositorio, ve a `Settings`.  
<img width="320" height="325" alt="image" src="https://github.com/user-attachments/assets/9c6776d9-6e6f-4d60-9938-00a317b7469e" />


Luego en el menú del lado izquierdo ve a `Deploy Keys`.  
<img width="1005" height="131" alt="image" src="https://github.com/user-attachments/assets/92bc9a4f-060f-47b6-9608-6a04593700d3" />


Dale a `Add deploy key`,  
<img width="1005" height="556" alt="image" src="https://github.com/user-attachments/assets/d1d3a026-2091-441f-a7be-c9bcd34cd74e" />  

y en el recuadro `Key` pega la clave que aparece dentro del archivo de llave pública `my_key1.pub` (puedes abrir el archivo con el Bloc de Notas, seleccionar todo `Ctrl+A` y luego copiar `Ctrl+C`) o en visual studio code en el archivo .pub.  
<img width="1115" height="237" alt="image" src="https://github.com/user-attachments/assets/3d3ad4d3-0ad3-4107-b8ec-4a658cea6eab" />

Bajo ninguna circunstancia debes pegar la clave privada que aparece en el archivo sin extensión `my_key1`. Finalmente, agrega algún título alusivo en el recuadro `Title` en GitHub, activa los permisos de escritura (Writing Access) en la casilla debajo del cuadro de la clave, y dale a guardar llave.  

<img width="999" height="573" alt="image" src="https://github.com/user-attachments/assets/fd146e2f-91b9-4515-95c3-b92728d28201" />

Al agregar la Key saldrá la siguiente ventana.  
<img width="1355" height="508" alt="image" src="https://github.com/user-attachments/assets/57b9e7f0-351c-417e-b3df-28a4d441c263" />



## Inciar sesión con SSH

Con las claves (pública y privada) y el archivo `config` en la carpeta correcta `~/.ssh` (o `C:\Users\your_user\.ssh` en Windows), y los permisos adecuados para la llave privada, ya se puede iniciar sesión mediane SSH. En Linux y WSL debes ejecutar:

```bash
ssh -T -F ~/.ssh/config git@github-host1
```

Y en Windows:

```Powershell
ssh.exe -T -F "C:\Users\your_user\.ssh\config" git@github-host1
```

En donde el parámetro `-T` indica que la conexión se efectúa sin el modo de (pseudo)terminal activado, lo cual es necesario en GitHub ya que no vamos a usar comandos de Bash en el servidor remoto (GitHub); el parámetro `-F ruta/a/config` es opcional y sirve para indicar la ruta del archivo `config`, si se omite toma por defecto la ruta `~/.ssh/config` en Linux nativo (pero parece que en Windows y WSL no, por lo que en Windows sí se debe escribir la ruta del `config`), y finalmente se escribe el nombre de usuario `git` (para hacer push en Git siempre se usa este nombre) seguido de un arroba y el host que especificaste en `config`.  
<img width="405" height="156" alt="image" src="https://github.com/user-attachments/assets/758cd7e9-b44e-42b4-8349-c49b06beab3f" />

Posteriormente el programa te pedirá que ingreses la passphrase con la que creaste la llave asignada al host, y con esto se iniciará sesión.

<img width="649" height="160" alt="image" src="https://github.com/user-attachments/assets/18a8a465-c983-41a5-b510-e477c1d191a7" />

## Conectando más de un usuario por computadora (opcional)

Si más de un usuario se requiere conectar a sus respectivos repositorios de GitHub en el mismo equipo, basta con repetir el comando del punto anterior, pero cambiando el nombre del host (después del arroba) por el host asignado a ese repositorio. Por ello es importante el archivo `config` con distintos Hosts. Por ejemplo, para conectar un segundo usuario:

```bash
ssh -T -F ~/.ssh/config git@github-host2
```

Y el programa solicitará al usuario ingresar su passphrase para `my_key2`. En Windows simplemente se agrega la terminación <b>.exe</b> al comando `ssh`, y se usa la ruta `"C:\Users\your_user\.ssh\config"` del `config` en Windows.


# Enlace entre repositorios Git y GitHub 

## Clonar un repositorio remoto

Para bajar un repositorio remoto para **colaborar** se debe bajar utilizando protocolo **SSH**. Para ello en el repositorio remoto se debe hacer lo siguiente:

   1. En la ventana del repositorio, click en `code`
   2. Se depliega la ventana **Local**, entrar a la pestaña `SSH`
   3. Copiar en el portapapeles el link, ejemplo:

      ```bash
      git clone git@github.com:LINX-ICN-UNAM/nombre-del-repositorio.git
      ```

<img src="img/git_clone.gif" alt="Consola Remota Password" />

Para poder clonar desde nuestra organización se debe usar una [llave SSH](GitHub.md#inciar-sesión-con-ssh) asociada al dispositivo que hace el `git clone` por lo que ya debe haber sido aprobada esa llave por algun administrador para utilizarla. Si aun no tienes aprobada tu llave o si desconoces del tema avisa a [Josué Rodríguez](https://github.com/alfjosue1997), [Fernando Caballero](https://github.com/Ferman333), o a [Addi Trejo](https://github.com/Additrejo).

 Con `github.com:user/repo.git` utilizarás el comando `git clone` en la terminal, ya sea **bash**, **wsl**, **git** o **powershell** pero deberas cambiar `github.com` por el host que quieras utilizar, de lo contrario aparecerá un error. Eso es debido a que el **Hostname** ya no es `@github.com` sino que deberas usar alguno de los alias que usaste en el archivo *.ssh*, ejemplo:

```bash
git clone git@github-host1:LINX-ICN-UNAM/nombre-del-repositorio.git
```

<img src="img/git_clone_ssh1.jpg" alt="Consola Remota Password" />

Con el host que ingresen les pedira la contraseña de la llave SSH asociada. Asi podrán clonar el repositorio remoto

<img src="img/git_clone_ssh2.jpg" alt="Consola Remota Password" />

Para poder bajar un repositorio remoto utilizando el protocolo **HTTP**:

```Bash
git clone https://github.com/TU-USUARIO/TU-REPO.git
```

## Subir un repositorio remoto

Lo que se esta haciendo es copiar todo el proyecto a la nube, en este caso a Github. Ahí se almacena de manera privada, publica o compartida

Para cargar un repositorio a GitHub varia un poco si empezaste desde `git init` o desde `git clone`

### Desde `init`

Si comenzaste desde `git init`, en GitHub **DEBES CREAR** un repositorio con el nombre de tu nuevo repositorio, de lo contrario git no sabrá adonde mandarlo.

Se abre la carpeta oculta `.git` del repositorio local de tu proyecto (esta carpeta se crea automáticamente al ejecutar `git init` o `git clone`), y luego se abre el archivo `config` de ese Git (NO CONFUNFIR con el `config` de <b>.ssh</b>). Ahora, se debe buscar la sección que contiene los datos del repositorio remoto, y edítala para que se vea como sigue:

```bash
[remote "origin"]
  url = github-host1:user/repo.git
  fetch = +refs/heads/*:refs/remotes/origin/*
```

En la línea de `url` se debe cambiar el `github-host1` por el nombre que asignaste a tu Host en SSH, y `user/repo.git` es el nombre del usuario propietario en GitHub seguido del nombre del repositorio donde estás trabajando. 

Tambien se puede hacer desde la terminal:

```bash
git remote add origin github-host1:user/repo.git
```

Para comprobar:

```bash
$ git remote -v
> origin  git@github-host1:user/repo.git (fetch)
> origin  git@github-host1:user/repo.git (push)
```

Opcional: Renombrar la rama principal a 'main'

```bash
git branch -M main 
```

Tras esto, cualquier push que se haga mediante Git será enviado a GitHub mediante el host personalizado `github-host1` creado en pasos anteriores, el cual ya tiene asignada la misma llave de acceso que cargaste en el repositorio remoto GitHub con DeployKeys. 

Ahora, ya se puede ejecutar el primer push, sólo ve a la carpeta del proyecto donde inicializaste Git, abre una terminal ahí mismo y ejecuta (es lo mismo en Windows que en Linux):

```bash
git push -u origin main
```

Si creaste la llave RSA con una passphrase, cada vez que hagas push se te pedirá de nuevo. Esto agrega una capa de seguridad para que usuarios no autorizados con acceso al equipo no intenten mandar código hacia este repositorio, o simplemente que no lo manden aquí por error confundiéndolo con su repositorio (en caso de haber más de una persona usando GitHub en el mismo equipo). 

El nombre del remoto "origin" y la rama de trabajo actual (en este ejemplo "main") son recordados por Git, por lo que en los siguientes  <i>push</i> ya no es necesario especificarlos, y podrías ejecutar simplemente:

```bash
git push
```

Aunque la passphrase te la pedirá siempre.

### Desde `clone`

Si comenzaste desde `git clone`. Lo mas probable es que tu archivo `config` ya las configuraciones hechas por defecto, por lo que para cargar una modificación al repositorio puedes hacer 

```bash
git push
```
Si no funciono, probablemente tengas que configurar el repositorio remoto `origin`:

```bash
git remote set-url <name> <newurl>
git remote set-url origin git@github-host:usuario/nuevo-repo.git
```
Es equivalente a usar `git branch`. La sintaxis básica es:
```bash
git branch (--set-upstream-to=<upstream>|-u <upstream>) [<branch-name>]
```
`<upstream>` considera que `<branch-name>` será el nombre de la rama. Sí `<branch-name>` no se especifica, then it defaults to the current branch.

Ejemplos:
```bash
git branch --set-upstream-to=origin main
git branch -u origin rama1
```



## Ramas en Github

Si quieres subir una nueva rama:

```bash
git push -u origin <nueva-rama>
```

> La `-u` no es necesaria sí ya has configurado `origin` y **Github**

O borrar una nueva rama:

```bash
git push origin --delete <nombre-rama>
```

Para poder revisar el estado de las ramas se puede utilizar `--prune` para purgar las ramas borradas del repositorio remoto y que no aparezcan al hacer `git status`.

```bash
git fetch --prune
```

Eso añade a `~/.gitconfig` las siguientes líneas:
```txt
[fetch]
        prune = true
```

> `~ = /home/<user>` significa que es una configuración a nivel de ***usuario*** 

Luego, para ver todas las ramas:
```bash
git branch -ar
```

# Colaboración entre multiples personas

Al estar en una organización, debemos de trabajar en equipo y para que se pueda llevar un historial de cambios de un proyecto se debe de entender el flujo de trabajo y los comandos asociados

## WorkFlow

## Fork

`Fork` es en Github el copiar todo un repositorio en tu cuenta. 

Para poder contribuir con repositorios en Github lo mejor es seguir un esquema de *Issues* y *Pull Request* para poder llevar un orden en la contribución del proyecto, evaluar posibles errores o conflictos en el desarrollo, etc. 



>No se recomienda *forkear* todos los repositorios de interes, puedes darle `⭐star` si quieres guardarlo

## Upstream



Debes configurar un remoto que apunte al repositorio ascendente en Git para sincronizar los cambios que realizas en una bifurcación con el repositorio original. Esto también te permite sincronizar los cambios en el repositorio original con la bifurcación.

[Se abre la carpeta oculta `.git`](#desde-init) del repositorio local de tu proyecto (esta carpeta se crea automáticamente al ejecutar `git init` o `git clone`), y luego se abre el archivo `config` de ese Git (NO CONFUNFIR con el `config` de <b>.ssh</b>). Ahora, se debe buscar la sección que contiene los datos del repositorio remoto, y edítala para que se vea como sigue:

```bash
[remote "origin"]
	url = git@github-host:colab-username/your_fork.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[remote "upstream"]
	url = git@github-host:organization/repo.git
	fetch = +refs/heads/*:refs/remotes/upstream/*
```

En la línea de `url` se debe cambiar el `github-host` por el nombre que asignaste a tu Host en SSH. Tambien se debe cambiar lo siguiente:

- `colab-username/your_fork.git` nombre del colaborador y el nombre del fork.
- `organization/repo.git` 

es el nombre del usuario propietario en GitHub seguido del nombre del repositorio donde estás trabajando. 

Tambien se puede hacer desde la terminal:
```bash
git remote add origin github-host:your_user/your_fork.git
git remote add upstream github-host:organization/repo.git
```

Verifica el nuevo repositorio ascendente que especificaste para tu bifurcación.

```bash
$ git remote -v
> origin    github-host:your_user/your_fork.git (fetch)
> origin    github-host:your_user/your_fork.git (push)
> upstream  github-host:organization/repo.git (fetch)
> upstream  github-host:organization/repo.git (push)
```