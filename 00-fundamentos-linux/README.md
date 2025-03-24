# Ejercicios

## Ejercicios CLI

**Antes de proceder a la resolución de ejercicios**

Para poder realizar los ejercicios en mi sistema operativo Windows en una shell de BASH, tras haber instalado Virtual Box y Vagrant, he seguido los siguientes pasos:

```shell
# Moverse al directorio en el que se encuentra el archivo Vagrantfile
cd C:\Users\anabo\Documents\Personal\Formacion\bootcamp-devops-lemoncode\00-fundamentos-linux
# Iniciar máquinas virtuales
vagrant up
# Conectarse a la máquina virtual Cliente
vagrant ssh ubuntu-client
```

Una vez dentro de la shell de BASH, he procedido a resolver las tareas.

### 1. Crea mediante comandos de bash la siguiente jerarquía de ficheros y directorios

#### Enunciado

```bash
foo/
├─ dummy/
│  ├─ file1.txt
│  ├─ file2.txt
├─ empty/
```

Donde `file1.txt` debe contener el siguiente texto:

```bash
Me encanta la bash!!
```

Y `file2.txt` debe permanecer vacío.

#### Resolución

```bash
# Crear directorio foo
mkdir foo
# Verificar que el directorio se ha creado correctamente
ls -l
# Moverse al directorio foo
cd foo
# Crear directorios vacíos
mkdir dummy empty
# Verificar que los directorios se han creado correctamente
ls
# Moverse al directorio dummy
cd dummy
# Crear ficheros vacíos
touch file{1,2}.txt
# Verificar que los ficheros se han creado correctamente
ls
# Añadir contenido al fichero file1.txt
echo 'Me encanta la bash!!' > file1.txt
# Verificar que el contenido se ha creado correctamente
cat file1.txt
```

### 2. Mediante comandos de bash, vuelca el contenido de file1.txt a file2.txt y mueve file2.txt a la carpeta empty

#### Enunciado

El resultado de los comandos ejecutados sobre la jerarquía anterior deben dar el siguiente resultado.

```bash
foo/
├─ dummy/
│  ├─ file1.txt
├─ empty/
  ├─ file2.txt
```

Donde `file1.txt` y `file2.txt` deben contener el siguiente texto:

```bash
Me encanta la bash!!
```

#### Resolución

```bash
# Moverse al directorio dummy desde foo
cd dummy
# Desde el directory dummy, volcar contenido de file1.txt en file2.txt
cat < file1.txt > file2.txt
# Verificar contenido de file2.txt
cat file2.txt
# Volver al directorio foo
cd ..
# Mover fichero file2.txt al directorio empty
mv dummy/file2.txt empty/
# Verificar que el fichero se ha movido correctamente
ls empty
```

### 3. Crear un script de bash que agrupe los pasos de los ejercicios anteriores y además permita establecer el texto de file1.txt alimentándose como parámetro al invocarlo

#### Enunciado

Si se le pasa un texto vacío al invocar el script, el texto de los ficheros, el texto por defecto será:

```bash
Que me gusta la bash!!!!
```

#### Resolución

Pasos iniciales:

```bash
# Moverse a la home del usuario
cd /home/vagrant
# Eliminar directorio creado previamente y su contenido
rm -r foo
# Crear un archivo de script
vim ejercicio3.sh
```

He añadido el siguiente contenido en el fichero `ejercicio3.sh`:

```bash
#!/bin/bash

# Crear directorio foo
mkdir foo
# Moverse al directorio foo
cd foo
# Crear directorios vacíos
mkdir dummy empty
# Moverse al directorio dummy
cd dummy
# Crear ficheros vacíos
touch file{1,2}.txt
# Añadir contenido al fichero file1.txt según parámetro, con valor por defecto si este no se especifica
echo ${1:-'Que me gusta la bash!!!!'} > file1.txt
# Volcar contenido de file1.txt en file2.txt
cat < file1.txt > file2.txt
# Volver al directorio foo
cd ..
# Mover fichero file2.txt al directorio empty
mv dummy/file2.txt empty/
```
He ejecutado los siguientes comandos:

```bash
# Asegurar que el contenido se ha guardado correctamente
cat ejercicio3.sh
# Ejecutar script
bash ./ejercicio3.sh
# Verificar estructura de ficheros
ls foo
# Verificar contenido ficheros
cat foo/dummy/file1.txt
> Que me gusta la bash!!!!
cat foo/empty/file2.txt
> Que me gusta la bash!!!!
```

### 4. Crea un script de bash que descargue el contenido de una página web a un fichero y busque en dicho fichero una palabra dada como parámetro al invocar el script

#### Enunciado

La URL de dicha página web será una constante en el script.

Si tras buscar la palabra no aparece en el fichero, se mostrará el siguiente mensaje:

```bash
$ ejercicio4.sh patata
> No se ha encontrado la palabra "patata"
```

Si por el contrario la palabra aparece en la búsqueda, se mostrará el siguiente mensaje:

```bash
$ ejercicio4.sh patata
> La palabra "patata" aparece 3 veces
> Aparece por primera vez en la línea 27
```

#### Resolución

Pasos iniciales:

```bash
# Moverse a la home del usuario
cd /home/vagrant
# Crear un archivo de script
vim ejercicio4.sh
```
He añadido el siguiente contenido en el fichero `ejercicio4.sh`:

```bash
#!/bin/bash

# Declarar variable global en script
TEST_URL="https://es.wikipedia.org/wiki/Solanum_tuberosum"

# Verificar si se pasa un argumento
if [[ $# -lt 1 ]]; then
  echo "Se necesita al menos un parámetro para ejecutar este script"
  exit 1
fi

# Obtener respuesta
response=$(curl -s --fail "$TEST_URL")

# Verificar si se obtiene una respuesta válida antes de continuar
# He tenido que añadir las líneas "nameserver 8.8.8.8" y "nameserver 8.8.4.4" en "/etc/resolv.conf" para que la conexión funcione, sino arroja el siguiente error
if [ $? -ne 0 ] || [ -z "$response" ]; then
  echo "Error: Could not connect to the URL ${TEST_URL}. Please check the URL or your internet connection."
  exit 1
fi

# Almacenar contenido en un fichero temporal
echo "$response" > tempUrlResponse.txt

# Contar número de ocurrencias de la palabra exacta
count=$(grep -wo "$1" tempUrlResponse.txt | wc -l)

# Extraer el número de línea de la primera ocurrencia
firstLine=$(grep -n "$1" tempUrlResponse.txt | head -n 1 | cut -d: -f1)

# Eliminar el archivo temporal
rm tempUrlResponse.txt

# Mostrar los resultados
if [[ "$count" -eq 0 ]]; then
  echo "No se ha encontrado la palabra \"$1\""
else
  echo "La palabra \"$1\" aparece $count veces"
  echo "Aparece por primera vez en la línea $firstLine" 
fi
```

He ejecutado los siguientes comandos:

```bash
# Asegurar que el contenido se ha guardado correctamente
cat ejercicio4.sh
# Ejecutar el script sin argumentos
bash ejercicio4.sh
> Se necesita al menos un parámetro para ejecutar este script
# Ejecutar el script sin resultados
bash ejercicio4.sh palabra-aleatoria
> No se ha encontrado la palabra "palabra-aleatoria"
# Ejecutar el script con varios resultados
bash ejercicio4.sh patata
> La palabra "patata" aparece 15 veces
> Aparece por primera vez en la línea 1144
```

### 5. OPCIONAL - Modifica el ejercicio anterior de forma que la URL de la página web se pase por parámetro y también verifique que la llamada al script sea correcta

#### Enunciado

Si al invocar el script este no recibe dos parámetros (URL y palabra a buscar), se deberá de mostrar el siguiente mensaje:

```bash
$ ejercicio5.sh https://lemoncode.net/ patata 27
> Se necesitan únicamente dos parámetros para ejecutar este script
```

Además, si la palabra sólo se encuentra una vez en el fichero, se mostrará el siguiente mensaje:

```bash
$ ejercicio5.sh https://lemoncode.net/ patata
> La palabra "patata" aparece 1 vez
> Aparece únicamente en la línea 27
```

#### Resolución

Pasos iniciales:

```bash
# Crear una copia del archivo del ejercicio anterior
cp ejercicio4.sh ejercicio5.sh
# Editar el archivo
vim ejercicio5.sh
```
He editado el contenido del fichero `ejercicio5.sh`:

```bash
#!/bin/bash

# Verificar que recibe los argumentos necesarios
if [[ $# -lt 2 ]]; then
  echo "Se necesitan al menos dos parámetros para ejecutar este script"
  exit 1
elif [[ $# -gt 2 ]]; then
  echo "Se necesitan únicamente dos parámetros para ejecutar este script"
  exit 1
fi

# Verificar si se obtiene una respuesta válida antes de continuar
response=$(curl -s --fail "$1") || { echo "Error: Could not connect to the URL $1. Please check the URL or your internet connection."; exit 1; }

# Contar número de ocurrencias de la palabra exacta
count=$(echo "$response" | grep -wo "$2" | wc -l)

# Extraer el número de línea de la primera ocurrencia
firstLine=$(echo "$response" | grep -n "$2" | head -n 1 | cut -d: -f1)

# Mostrar los resultados
if [[ "$count" -eq 0 ]]; then
  echo "No se ha encontrado la palabra \"$2\""
elif [[ "$count" -eq 1 ]]; then
  echo "La palabra \"$2\" aparece $count vez"
  echo "Aparece únicamente en la línea $firstLine" 
else
  echo "La palabra \"$2\" aparece $count veces"
  echo "Aparece por primera vez en la línea $firstLine" 
fi
```

He ejecutado los siguientes comandos:

```bash
# Asegurar que el contenido se ha guardado correctamente
cat ejercicio5.sh
# Ejecutar el script sin argumentos
bash ejercicio5.sh
> Se necesitan al menos dos parámetros para ejecutar este script
# Ejecutar el script con más de 2 argumentos
bash ejercicio5.sh https://lemoncode.net/ patata 27
> Se necesitan únicamente dos parámetros para ejecutar este script
# Ejecutar el script sin resultados
bash ejercicio5.sh https://lemoncode.net/ patata
> No se ha encontrado la palabra "patata"
# Ejecutar el script con un único resultado
bash ejercicio5.sh https://es.wikipedia.org/wiki/Solanum_tuberosum 1885
> La palabra "1885" aparece 1 vez
> Aparece únicamente en la línea 1617
# Ejecutar el script con varios resultados
bash ejercicio5.sh https://es.wikipedia.org/wiki/Solanum_tuberosum patata
> La palabra "patata" aparece 15 veces
> Aparece por primera vez en la línea 1144
```