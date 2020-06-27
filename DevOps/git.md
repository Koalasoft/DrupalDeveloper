GIT
===
```
Instalación en sistemas debian, ubuntu
  $ apt-get install git

Configuración de usuario
  $ git config --global user.name usuario
  $ git config --global user.email usuario@correo.com
  $ git config --global core.editor emacs
  $ git config --list

Configuración estilo de fin de línea para linux-mac: Auto modifica CTRL (Windows) a LF (Unix)
  $ git config --global core.autocrlf input

Generar las llaves pública y privada
  $ ssh-keygen -t rsa -C usuario@correo.com

Ignorar archivos o carpetas para el versionamiento
  1. Crear un archivo .gitignore en la rais de la carpeta versionada
  2. Escribir en el archivo .gitignore los archivos y carpetas a ignorar. Por ejemplo
  .idea/
  .gitignore

Perder todos los cambios que no fueron commiteados
  $ git checkout -f

Crear una rama o branch nueva
  $ git branch crazy-experiment
  $ git checkout crazy-experiment
  ... codificar cambios ...
  $ git add -A
  $ git commit -m "message..."
  $ git push origin crazy-experiment

Agregar y Quitar archivos (Equivalente git add, git rm juntos)
  $ git add -u

Añadir porciones de cambios dentro de cada archivo al stage de cambios
  $ git add -p

Quitar archivos
  $ git rm

Cambiar de nombre una rama
  $ git branch -m <oldname> <newname>
  # Si estas en la rama que deseas cambiar de nombre
  $ git branch -m <newname>

Deshacer cambios del pull mas reciente
  $ git reset --hard

Retrocedemos al último commit y perdemos todos los cambios hechos.
  $ git reset --hard HEAD~1

Retrocedemos a el último commit y no perdemos los cambiosn hechos; apareceran pendientes de hace commit
  $ git reset --soft HEAD~1

Cancelar commit (antes de hacer push)
  $ git reset --hard HEAD~1

Ver cambios de un commit
  $ git show <id_commit>

Visualizar una rama existente
  $git checkout crazy-experiment

Ver el historial gráficamente
  $ git log --graph --full-history --all --pretty=format:"%h%x09%d%x20%s"

Ver el historial de un archivo
  $ git log /ruta/archivo

Ver el historial de un usuario
  $ git log --author="vacho"

Ver el historia de cada línea de codigo de un archivo
  $ git blame /ruta/archivo

Ver el historial de un trozo de código
git log --all -p -STrozo_de_codigo

Mezclar una rama con el master
  $ git checkout master
  $ git merge crazy-experimient

Agregamos una rama delante del master
  $ git checkout master
  $ git rebase crazy-experimient

Eliminar un branch
  $ git branch -D crazy-experiment
  $ git push origin :crazy-experiment

Ver aportes de líneas a un archivo
  $ git blame /ruta/archivo

Ver cambios en un archivo
  $ git diff /ruta/archivo

Ver cambios entre 2 commits
  $ git diff --name-only SHA1 SHA2

Arreglar el último commit en lugar de un nuevo commit
  # editamos hola.py y main.py
  $ git add hola.py
  $ git commit
  # añadir los cambios de main.py en el último commit
  $ git add main.py
  $ git commit --amend --no-edit

Hacer merge de un específico commit de una rama en otra rama (donde se esta trábajando)
  $ git cherry-pick <codigo_del_commit>

Lista de los commits que tienen un texto
  $ git log --all --grep='<texto>'
  
Restaurar un archivo borrado que no ha sido commiteado
  Restaura el estado del archivo en index
  $ git reset -- <ruta completa archivo>
  Salida del cambio hecho en el archivo en index
  $ git checkout -- <ruta completa archivo>

Ignorar un archivo en el repositorio sin .gitignore
  $ git update-index --skip-worktree /sites/default/settings.php

```
Trabajar como fork de un repositorio y actualizar
===
```
# Añadir repositorio remoto, llamarlo "upstream":
git remote add upstream https://github.com/whoever/whatever.git

# Extraer todos los branchs
git fetch upstream

# Estar seguro de que se está en el branch master
git checkout master

# Reescribir todos los commits
git rebase upstream/master
```

Montar repositorio para trabajar en remoto
===
```
# En la carpeta a versionar:
$ git init

//habilitamos la posibilidad de hacer push
$ git config receive.denyCurrentBranch ignore

//Git manejará los finales de línea linux-windows
$ git config --global core.autocrlf true
$ git config --global core.whitespace cr-at-eol

$ git add .
$ git commit -m "inicio"
$ git checkout -f

// Evitar hacer git checkout -f todo el tiempo
$ cd .git/hooks
$ vim post-receive
// Colocar las siguientes 2 líneas
#!/bin/sh
GIT_WORK_TREE=../ git checkout -f
$ chmod +x post-receive

# Para clonar
$ git clone ssh://nombre_cuenta@ip_servidor:puerto/ruta_a_la_carpeta

```
Ignorar archivos de un repositorio
===
```
// Crear un .gitignore  con la ruta relativa a las carpetas o archivos a omitir
$ git rm -r --cached .
$ git add .
$ git commit -m "reiciniamos git"
```

REFERENCIAS
---
https://www.atlassian.com/git/tutorials/merging-vs-rebasing/conceptual-overview
Hostgator
http://www.codigogratis.com.ar/como-utilizar-git-con-hostgator/
Fin de línea
https://stackoverflow.com/questions/5834014/lf-will-be-replaced-by-crlf-in-git-what-is-that-and-is-it-important