## Usos practicos de git

* Inicializamos un repositorio para comenzar a trabajar

`git init`

* Agregamos un archivo al area de staging

`git add README.md`

* Hacemos un commit local en el repo inicializado anteriormente

`git commit -m "mensaje del commit"`

* Nos aseguramos que el branch donde estemos parados se llame main, en caso de no ser así el siguiente comando renombra el branch donde estemos parado como main

`git branch -M main`

* Agregamos un repositorio remoto a nuestro repositorio local para subir lso cambios locales al servidor remoto

`git remote add origin http://[server]/[repository-name].git`

* Actualizamos el repositorio remoto con los cambios locales

`git push -u origin main`

* Agregar solo una parte de un archivo al area de staging para el proximo commit

`git add -p <filename>`

Git dividirá el archivo en lo que considere "trozos" sensatos (porciones del archivo). A continuación, le hará esta pregunta:

    `Stage this hunk [y,n,q,a,d,s,e,?]?`

Estas son algunas opciones sacadas del help (lo dejo en ingles porque se va a entender mejor que si lo traduzco):

[ y ] - stage this hunk for the next commit

[ n ] - do not stage this hunk for the next commit

[ q ] - quit; do not stage this hunk or any of the remaining hunks

[ a ] - stage this hunk and all later hunks in the file

[ d ] - do not stage this hunk or any of the later hunks in the file

[ s ] - split the current hunk into smaller hunks

[ e ] - manually edit the current hunk (You can then edit the hunk manually by replacing +/- by #)

[ ? ] - print hunk help

* Para poder verificar los cambios que tenemos en el stage podemos usar

`git diff --staged`

* Para quitar los hunk/"trozos" agregados por error al stage

`git reset -p`

* Para sacar un archivo de Staging (esto saca el archivo del area de stage para que no forme parte del proximo commit)

`git restore --staged myFile`

En caso de querer descartar también los cambios locales en este archivo, eliminar la opción --staged

* Para borrar un branch LOCAL

`git branch -d <branch>`

La opción -d elimina el branch únicamente si este esta sincronizado con el branch remoto. Utilizar -D si para forzar la eliminación de un branch, incluso si aún este no ha sido sincronizado aún con el remoto.

* Para borrar un branch REMOTO

`git push <remote> --delete <branch>`

También puedes utilizar este comando corto para borrar

`git push <remote> :<branch>`

* Otra forma de borrar branch locales

`git fetch -p`

La flag -p significa "prune". Después de hacer el fetching, los branches que ya no existan en el repositorio remoto serán eliminados en el repositorio local.

## Configurar credenciales por repositorio

Si necesitas usar una identidad distinta solo en el proyecto actual, ejecuta los siguientes comandos dentro del repositorio para sobreescribir la configuración global:

```bash
git config user.name "<nombre>"
git config user.email "<correo>"
```

Opcionalmente puedes guardar las credenciales para este repositorio mediante un helper, por ejemplo:

```bash
git config credential.helper store
```

Para verificar los valores configurados de forma local utiliza:

```bash
git config --list --local
```

