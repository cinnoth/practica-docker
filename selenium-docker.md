## Instalación de Angular y Selenium 
1. **Descargar la imagen de Ubuntu.** Ejecuta el siguiente comando para descargar la imagen oficial de Ubuntu:
    
    ```
   docker pull ubuntu
   ```

2. **Crear y acceder al contenedor.** Ejecuta el siguiente comando para crear y entrar en un contenedor interactivo basado en Ubuntu, en este caso se expondrá el puerto 4200 para su uso posterior con Angular:

    ```
   docker run -it -p 4200:4200 --name ubuntu-container ubuntu bash
    ```

    📌 Explicación:

    - docker run → Crea y ejecuta un nuevo contenedor
    - -it → Modo interactivo (para usar la terminal dentro del contenedor).
    - -p 4200:4200 → Mapea el puerto 4200 del contenedor al puerto 4200 del host.
    - --name ubuntu-container → Asigna el nombre ubuntu-container al contenedor.
    - ubuntu → Es la imagen base que se usará.
    - bash → Abre la terminal dentro del contenedor.

3. **Instalar Node.js y herramientas necesarias dentro del contenedor.** Una vez dentro del contenedor, instala lo que necesitas:

    ```
    apt update

    apt upgrade

    apt install nano

    apt install curl

    apt install wget

    apt install git

    apt install nodejs

    apt install npm
    ```


    *📃 NOTA: puedes instalarlos uno a uno o usar:*

    ```
   apt update && apt upgrade -y
    ```

    ```
    apt install -y nano curl wget git nodejs npm
   ```

4. **Instalar Angular CLI.** Ejecuta este comando dentro del contenedor:

    ```
   npm install -g @angular/cli
    ```
5. **Crear un nuevo proyecto Angular.** Navega a una carpeta "angular-selenium-docker" donde quieres crear el proyecto y ejecuta:

    ```
    ng new proyecto-angular-selenium
    ```
6. **Entrar a la carpeta del proyecto:**

    ```
    cd proyecto-angular-selenium
    ```

7. **Instalar Selenium y WebDriver.** Ejecuta los siguientes comandos para instalar Selenium en tu proyecto Angular:

    ```
    npm install selenium-webdriver chromedriver --save-dev
    ```
8. **Instalar Chromedriver y Chrome:**

    1. Instala Chromedriver en el contenedor con el siguiente comando:

        ```
        npm install chromedriver --save-dev
        ```

    2. Verifica si se instaló correctamente:
        ```
        npx chromedriver --version
        ```

    3. Instala Chrome con los siguientes comandos:
        ```
        apt update
        ```
        ```
        apt install -y wget curl
        ```
        ```
        wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add -
        ```
        ```
        echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" | tee /etc/apt/sources.list.d/google-chrome.list
        ```
        ```
        apt update
        ```
        ```
        apt install -y google-chrome-stable
        ```
    
    4. Luego, verifica que se haya instalado correctamente con:
        ```
        google-chrome --version
        ```
9. **Crear una carpeta test** con archivo de prueba con Selenium. Crea un archivo title-test.js en una carpeta tests/. 
    ```
    mkdir tests
    cd tests
    touch title-test.js
    ```

10. En el archivo title-test.js escribe el caso de prueba, por ejemplo: 
    ```
    const { Builder, By, until } = require('selenium-webdriver');
    const chrome = require('selenium-webdriver/chrome');

    (async function example() {
    let options = new chrome.Options();
    
    // Generar un directorio de usuario único
    const uniqueUserDir = `/tmp/chrome-profile-${Date.now()}`;
    options.addArguments(`--user-data-dir=${uniqueUserDir}`);

    options.addArguments('--headless'); // Ejecutar sin interfaz gráfica
    options.addArguments('--disable-gpu');
    options.addArguments('--no-sandbox');
    options.addArguments('--disable-dev-shm-usage');

    let driver = new Builder()
        .forBrowser('chrome')
        .setChromeOptions(options)
        .build();

    try {
        await driver.get('http://localhost:4200');
        await driver.wait(until.elementLocated(By.css('app-root h1')), 5000);
        let title = await driver.findElement(By.css('app-root h1')).getText();
        console.log('✅ Título de la página:', title);
    } catch (error) {
        console.error('Error en la prueba:', error);
    } finally {
        await driver.quit();
    }
    })();
    ```
11. **Ejecutar la prueba.** Ejecuta la prueba para comprobar que el test se corre correctamente. 

    1. Asegúrate de que tu aplicación Angular esté corriendo:

        ```
        ng serve --host 0.0.0.0 --port 4200
        ```
    *NOTA: es muy importante que tu proyecto Angular esté corriendo antes de correr tus test*

    2. Luego, ejecuta la prueba con (o dependiendo de la ubicación de tu test):

        ```
        node tests/title-test.js
        ```

    *Si tus test no te corren verifica que estes corriendo tu aplicación Angular. Ejecuta dentro del contenedor:*
        ```
        curl -I http://localhost:4200
        ```

    - Si responde algo como 200 OK, significa que la aplicación está corriendo.
    - Si dice Connection refused, entonces la aplicación no está corriendo o está corriendo en otro
    - Si Angular no está corriendo, intenta iniciarlo manualmente con:

        ```
        ng serve --host 0.0.0.0 --port 4200
        ```

## CREA UN SCRIPT para ejecutar el proyecto angular y las pruebas.


1. En la carpeta realizada anteriormente de nombre "angular-selenium-docker" crea un script llamado run-angular-test usando nano.

    ```
    #!/bin/bash

    # Configuración de rutas
    PROJECT_PATH="/angular-selenium-docker/proyecto-angular-selenium"
    SELENIUM_TESTS_PATH="/angular-selenium-docker/proyecto-angular-selenium/tests"

    # Verificar si Angular CLI está instalado
    if ! command -v ng &> /dev/null; then
        echo "Error: Angular CLI no está instalado o no está en el PATH"
        exit 1
    fi

    # Navegar al directorio del proyecto
    echo "Iniciando el proyecto de Angular..."
    cd "$PROJECT_PATH" || { echo "Error: No se encontró la carpeta del proyecto"; exit 1; }

    # Levantar servidor Angular en segundo plano y guardar logs
    ng serve --host 0.0.0.0 --port 4200 > angular.log 2>&1 &
    ANGULAR_PID=$!
    echo "Angular levantado con PID $ANGULAR_PID"

    # Capturar señal para matar el proceso de Angular al salir
    trap 'kill $ANGULAR_PID' EXIT

    # Esperar a que el servidor de Angular esté disponible
    echo "Esperando a que Angular se levante..."
    while ! curl -s http://localhost:4200 > /dev/null; do
        sleep 2
    done

    # Ejecutar pruebas de Selenium
    echo "Ejecutando pruebas Selenium..."
    cd "$SELENIUM_TESTS_PATH" || { echo "Error: No se encontró la carpeta de pruebas Selenium"; exit 1; }
    node title-test.js

    echo "Pruebas finalizadas."

    # Esperar a que el servidor de Angular termine
    wait $ANGULAR_PID
    echo "Angular detenido. Proceso completado."
    ```
2. **Ejecutar el menú**
    - Dar permisos de ejecución Para poder ejecutar el script, debes darle permisos de ejecución. Ejecuta el siguiente comando en la terminal:
        
        ```
      chmod +x run-angular-test.sh
        ```

    - Ejecutar el script Para ejecutar el script, simplemente escribe el siguiente comando en la terminal:

        ```
      ./run-angular-test.sh
        ```