# Práctica en Docker

## 1️⃣Instalación de Docker con Windows
1. Instalar [Docker Desktop](https://www.docker.com/) en la pagina oficial de Docker
2. Seguir la [documentación oficial de Docker](https://docs.docker.com/get-started/get-docker/) para la instalación o si se prefiere, seguir el tutorial de [Cómo instalar Docker en Windows ](https://www.youtube.com/watch?v=AxBQFrHK35c)

## 2️⃣Configuración de una máquina virtual Ubuntu con Docker

En esta sección se aborda la configuración de un contenedor Ubuntu con Node.js y la creación de un archivo holamundo.js como práctica

1. **Descargar la imagen de Ubuntu.** Ejecuta el siguiente comando para descargar la imagen oficial de Ubuntu:
    
    ``
   docker pull ubuntu
   ``

3. **Crear y acceder al contenedor.** Ejecuta el siguiente comando para crear y entrar en un contenedor interactivo basado en Ubuntu, en este caso se expondrá el puerto 4200 para su uso posterior con Angular:

    ``docker run -it -p 4200:4200 --name ubuntu-container ubuntu bash``

    📌 Explicación:

    - docker run → Crea y ejecuta un nuevo contenedor
    - -it → Modo interactivo (para usar la terminal dentro del contenedor).
    - -p 4200:4200 → Mapea el puerto 4200 del contenedor al puerto 4200 del host.
    - --name ubuntu-container → Asigna el nombre ubuntu-container al contenedor.
    - ubuntu → Es la imagen base que se usará.
    - bash → Abre la terminal dentro del contenedor.

4. **Instalar Node.js y herramientas necesarias dentro del contenedor.** Una vez dentro del contenedor, instala lo que necesitas:

    ``apt update``

    ``apt upgrade``

    ``apt install nano``

    ``apt install curl``

    ``apt install wget``

    ``apt install git``

    ``apt install nodejs``

    ``apt install npm``


    *📃 NOTA: puedes instalarlos uno a uno o usar:*

    ``apt update && apt upgrade -y``

    ``apt install -y nano curl wget git nodejs npm``

5. **Crear un archivo index.js con "Hola Mundo".** 
    1. Dentro del contenedor, usa el editor nano para crear un archivo.

        ``nano holamundo.js``

    2. Escribe el siguiente código en nano

        ``console.log("¡Hola, mundo desde un contenedor Ubuntu en Docker! :)");``
    
    3. Guarda y cierra nano (presiona Ctrl + X, luego Y y Enter).

6. **Ejecutar el código JavaScript**
    1. Ejecuta el script dentro del contenedor:

        ``node holamundo.js``

        *Si todo está bien, verás el mensaje: ¡Hola, mundo desde un contenedor Ubuntu en Docker!*

7. **Salir del contenedor de Ubuntu.** Para salir del contenedor de Ubuntu, ejecuta el siguiente comando:

    ``exit``

8. **Guardar los cambios en una nueva imagen.** Una vez que hayas realizado las configuraciones necesarias, puedes guardar los cambios en una nueva imagen de Docker. Para ello, primero detén el contenedor:

    ``docker stop ubuntu-container`

    Luego, guarda los cambios en una nueva imagen:

    ``docker commit ubuntu-container2 ubuntu-custom2``

    📌 Explicación:

    - ubuntu-container: Nombre del contenedor actual.
    - ubuntu-custom: Nombre de la nueva imagen.

## 3️⃣Pasos para crear un proyecto Angular dentro del contenedor creado.

1. **Asegurar de estar dentro del contenedor**
    - Si ya tienes tu contenedor corriendo, accede a él con:

        ``docker exec -it ubuntu-container bash``

    - Si el contenedor está detenido, inícialo antes:

        ``docker start ubuntu-container``
        ``docker exec -it ubuntu-container bash``

2. **Crear una carpeta** para guardar el proyecto Angular con nombre "proyecto-angular-docker", el cual posteriormente será usado para guardar un menú con bash. Asegurate de usar mkdir -nombre y acceder a ella mediante cd

3. **Instalar Angular CLI.** Ejecuta este comando dentro del contenedor:

    ``npm install -g @angular/cli``

4. **Crear un nuevo proyecto Angular.** Navega a una carpeta donde quieres crear el proyecto y ejecuta:

    ``ng new mi-proyecto-angular``

5. **Ejecutar el proyecto Angular.** Una vez creado el proyecto, entra a la carpeta del proyecto:

    ``cd mi-proyecto-angular``

6. **Inicia el servidor de desarrollo:** 

    ``ng serve --host 0.0.0.0 --port 4200``

    📌 Explicación: El parámetro --host 0.0.0.0 permite que el servidor Angular sea accesible fuera del contenedor.

## 4️⃣Crear menú interactivo en Bash
1. En la carpeta realizada anteriormente de nombre "proyecto-angular-docker" crea un script llamado menu-angular.sh usando nano

```
#!/bin/bash

#Ruta del proyecto angular
ANGULAR_PATH="/proyecto-angular-docker/mi-proyecto-angular"

#fx para ejecutar angular desde bash
run_angular(){
        echo "Iniciando el servidor Angular..."
        cd "$ANGULAR_PATH" || {echo "Error: no se encontro directorio del proyecto"}
        ng serve --host 0.0.0.0 --port 4200
}


while true; do
        echo "***** MENU DE ANGULAR *****"
        echo "1. Ejecutar proyecto Angular"
        echo "2. Ejecutar pruebas unitarias"
        echo "3. Salir"
        echo -n "Elige una opcion: "
        read opcion

        case $opcion in

        1)echo "Has seleccionado la Opci  n 1."
            run_angular ;;

        2)
            echo "Has seleccionado la Opci  n 2."
            read -p "Presiona Enter para volver al men  ..."
            ;;
        3)
            echo "Saliendo..."
            exit 0
            ;;

        *)
            echo "Opci  n inv  lida, intenta de nuevo."
            read -p "Presiona Enter para continuar..."
            ;;
    esac
done
```
*📃NOTA: Este script crea un menú interactivo en Bash con tres opciones: Ejecutar el proyecto Angular, Ejecutar pruebas unitarias y salir del menú. Puedes personalizar las opciones y acciones según tus necesidades.*

2. **Ejecutar el menú**
    - Dar permisos de ejecución Para poder ejecutar el script, debes darle permisos de ejecución. Ejecuta el siguiente comando en la terminal:
        
        ``chmod +x menu-angular.sh``

    - Ejecutar el script Para ejecutar el script, simplemente escribe el siguiente comando en la terminal:

        ``./menu-angular.sh``
