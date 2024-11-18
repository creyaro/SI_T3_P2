
# Práctica 2: Gestión de usuarios, grupos y permisos en Linux

En esta práctica aprenderemos sobre la gestión de usuarios, grupos y permisos en Linux, así como el uso de nuevas herramientas de cracking como **John the Ripper** y **Hashcat**. 

---

## 💻 1. Instalación del entorno

### 🐋 Docker
**Docker** es una plataforma que permite empaquetar aplicaciones y sus dependencias en contenedores, asegurando que se ejecuten de manera consistente en cualquier entorno. A diferencia de las máquinas virtuales (VM), que emulan un sistema operativo completo, los contenedores de Docker comparten el núcleo del sistema operativo subyacente, lo que los hace mucho más ligeros y rápidos. Docker facilita la creación, el despliegue y la gestión de aplicaciones de manera eficiente, permitiendo que los desarrolladores creen entornos aislados para ejecutar sus aplicaciones sin preocuparse por las diferencias entre máquinas. En Docker hay dos conceptos básicos que es importante conocer:

- **Imagen**: Es un paquete inmutable que contiene todo lo necesario para ejecutar una aplicación, incluyendo el código, las bibliotecas, las dependencias y la configuración. Las imágenes son la base para crear los contenedores y se pueden compartir entre diferentes usuarios y entornos.

- **Contenedor**: Es una instancia en ejecución de una imagen, y proporcionan un entorno aislado donde se ejecuta la aplicación. 

### Instalación de Docker
1. **Windows**: Descarga Docker Desktop desde [docker.com](https://www.docker.com) e instálalo.
2. **Linux**: Instala Docker usando los siguientes comandos:

   ```bash
   sudo apt update
   sudo apt install docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

### Configuración del entorno
1. Descarga la imagen de Docker preparada para esta práctica:
   ```bash
   docker pull creyaro/si-practica-2:v1
   ```

2. Inicia un contenedor con la imagen descargada:
   ```bash
   docker run -it --rm creyaro/si-practica-2:v1
   ```

3. Ahora estarás dentro del contenedor y del entorno donde realizaremos la práctica. Como puedes ver, el punto de partida es el usuario `usuario`.

🚪Para salir del contenedor basta con ejecutar `exit`.

---

## 🛠️ 2. Comandos de Linux

### Gestión de usuarios
- Crear un usuario: `sudo useradd nombre_usuario`
- Crear un usuario con directorio personal (/home/nombre_usuario): `sudo useradd -m nombre_usuario`
- Asignar una contraseña a un usuario: `sudo passwd nombre_usuario`
- Cambiar de usuario: `su nombre_usuario`
- Iniciar una consola bash: `bash`

### Gestión de grupos
- Crear un grupo: `sudo groupadd nombre_grupo`
- Añadir un usuario a un grupo: `sudo usermod -aG nombre_grupo nombre_usuario` (solo root)
- Ver los grupos a los que pertenece un usuario: `id nombre_usuario`
- Ver los usuarios y grupos del sistema (los usuarios con **ID > 1000** son los usuarios creados manualmente): `cat /etc/group`

### Permisos de archivos

Previamente tenemos que recordar que el propietario de un archivo tiene control total sobre él, incluyendo la capacidad de cambiar permisos y eliminarlo. Un usuario con permisos tiene acceso limitado (lectura, escritura o ejecución) al archivo según los permisos otorgados, pero no puede modificar la propiedad ni cambiar los permisos del archivo.


- Cambiar permisos: `chmod permisos archivo`. Por ejemplo:
   - ``chmod 700 archivo.txt``:
Establece permisos de lectura, escritura y ejecución solo para el propietario del archivo, mientras que otros usuarios no tienen acceso.

   - ``chmod 777 archivo.txt``:
Otorga lectura, escritura y ejecución a todos los usuarios (propietario, grupo y otros), lo que hace que el archivo sea accesible por cualquiera.

- Cambiar propietario: `sudo chown usuario archivo`
- Cambiar grupo propietario: `sudo chgrp grupo archivo`
- Ver permisos de un archivo: `ls -l archivo`

### El comando chmod

El comando ``chmod`` se utiliza para cambiar los permisos de acceso a archivos y directorios. El formato numérico de chmod usa tres dígitos, cada uno representando los permisos para diferentes grupos de usuarios: propietario, grupo y otros usuarios.

#### Cómo funciona
- Primer número: Permisos para el propietario del archivo.
- Segundo número: Permisos para el grupo del archivo.
- Tercer número: Permisos para otros usuarios.

Cada número es la suma de los permisos:

- 4 = lectura (r)
- 2 = escritura (w)
- 1 = ejecución (x)

### Otros comandos

- `find /folder -name "file.txt"`: Buscar archivos en una ubicación y con un nombre dado.
- `cat file.txt`: Visualizar el contenido de un fichero.
- `cat file.txt | grep line`: Visualizar el contenido de un fichero, pero solo las líneas que coincidan con el parámetro que se le pasa al grep.

## 🔓 3. Herramientas de descifrado

### John the Ripper
John the Ripper es una herramienta para descifrar contraseñas mediante ataques de fuerza bruta o diccionario. Soporta múltiples formatos de hash, como MD5.

#### Uso básico
1. Crear un archivo con el hash a descifrar:
   ```bash
   echo "nombre_hash:hash_encriptado" > hash.txt
   ```
2. Ejecutar John the Ripper en el archivo:
   ```bash
   john --format=raw-md5 hash.txt
   ```
3. Ver la contraseña descifrada:
   ```bash
   john --show hash.txt
   ```

### Hashcat
Hashcat es una herramienta avanzada de descifrado de contraseñas que aprovecha la GPU para mejorar el rendimiento.

#### Uso básico
1. Ejecutar Hashcat sobre un hash:
   ```bash
   hashcat -m 0 -a 3 [máscara] hash.txt
   ```

    - `-m`: Especifica el tipo de hash (ej., 0 para MD5).
    - `-a`: Define el modo de ataque (ej., 3 para fuerza bruta, 0 para uso de diccionarios).

    La máscara es opcional y define el patrón a usar para descifrar la contraseña. Las opciones son:

    - ``?l``: Letra minúscula (a-z)
    - ``?u``: Letra mayúscula (A-Z)
    - ``?d``: Dígito (0-9)
    - ``?s``: Carácter especial (como @, #, $)
    - ``?a``: Cualquier carácter alfanumérico (?l, ?u, ?d, ?s)
    - ``?b``: Todos los caracteres imprimibles (incluye espacios)

    Por ejemplo, para una contraseña que empieza con un carácter especial seguido de dos dígitos y tres letras minúsculas el comando sería:

    ```
    hashcat -m 0 -a 3 ?s?d?d?l?l?l hash.txt
    ```


2. Si queremos usar diccionarios, el comando sería el siguiente:

   ```
   hashcat -m 0 -a 0 hash.txt diccionario.txt 
   ```

💡 Si hashcat termina pero no muestra la contraseña encontrada, es suficiente con añadir la opción `--show` al comando anterior.

## 🕹️ 4. Objetivo de la práctica

Analiza el sistema (usuarios, ficheros, permisos, etc.) y realiza las acciones indicadas en *alguna parte* del entorno.

Crea un documento PDF donde expliques las acciones que has realizado y por qué, adjuntando capturas de los comandos ejecutados y su salida.
