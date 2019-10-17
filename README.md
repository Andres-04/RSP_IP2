# Sistema de monitoreo basado en Iot para aplicaciones en interiores usando un controlador de dos niveles

A modo de afianzar los conocimientos obtenidos durante el segundo corte se llevó acabo el siguiente proyecto. En este proyecto pusimos en practica conocimientos como crear una red HOSTPOT desde la Raspberry PI 3, protocolo I2C y utilizamos conocimientos previos como el manejo y configuración del sensor digital DS18B20.

# Objetivos

- Afianzar los conceptos básicos asociados a Internet de las Cosas (IoT).
- Familiarizarse con el manaje de gestores de bases de datos y servidores Web en sistemas embebidos.
- Familiarizarse con una arquitectura de dos niveles compuesta por un controlador de bajo nivel y un sistema embebido para la monitorización de temperatura.
- Desarrollar un controlador de dos niveles para la monitorización de temperatura utilizando la plataforma Raspberry Pi, un microcontrolador (Arduino), el sensor digital de temperatura DS18B20, una base de datos e interface Web local y remota.
- Implementar un servidor central en la plataforma Raspberry Pi compuesto al menos por un servidor web y una base de datos (gestionada en SQLite).
- Implementar un controlador de bajo para la lectura del sensor DS18B20 y transferencia de las mediciones al servidor central.
- Proponer y codificar un programa para la adquisición, almacenamiento, procesamiento y visualización de los valores arrojados por el controlador de bajo utilizando scripts.
- Presentar y sustentar la solución alcanzada.
- Elaborar un informe del proyecto empleando el formato IEEE.

# Materiales utilizados

- 1 PC
- Chrome/Mozilla/Internet Explorer
- Acceso a servicio de Internet
- 1 actuador LED
- 1 Raspberry Pi
- 1 sensor digital DS18B20
- 1 protoboard y cables
- Elementos eléctricos para el circuito de acondicionamiento del sensor DS18B20 y actuador LED

# Proceso de implementación

- Configuración de la comunicación I2C

Estos pasos solo son validos para Raspbian, revisa el SO que estas utilizando ya que hay unos que vienen listos para usar el I2C, de cualquier manera se recomienda actualizar la versión del SO que se este utilizando. Para comenzar abrimos la terminal y escribimos el siguiente comando:

Ejecutar desde Terminal:

`sudo raspi-config`

Ahora seguimos las siguientes opciones:

Seleccionar `8.- “Advanced Options”`

Seleccionar `A7 “I2C”`

Elegir `“YES”`

En la venta que muestra el mensaje `“The ARM I2C interface is enabled”`

Seleccionar `“OK”`

La siguiente ventana nos pregunta que si queremos cargar el módulo I2C por default

Seleccionar `“Yes”`

En la ventana que nos indica que el módulo I2C ha sido habilitado

Seleccionar `“OK”`

Seleccionar `“Finish”`

Luego de finalizar los pasos anteriores, procedemos a escribir los siguienes comandos en la consola de nuestra Raspberry PI:

Ejecutamos en terminal:

`sudo nano /etc/modules`

Agregamos las siguientes lineas al archivo:

```
i2c-bcm2708

i2c-dev
```

Para guardar cambios presionamos `ctr + x`, seguido de Y y enter.

Para que podamos usar la interfaz I²C desde Python debemos instalar las librerías correspondientes. Ejecutamos en terminal:

`sudo apt-get update`

`sudo apt-get install –y python-smbus i2c-tools`

Para que los cambios se apliquen reiniciamos el sistema

`Sudo reboot`

Ya tenemos habilitado el módulo de comunicación I²C en Raspberry.

Los siguientes pasos son opcionales y tienen como objetivo escanear algún dispositivo I²C conectado a la Raspberry.

Después de reiniciar la tarjeta podemos verificar que el modulo I²C se esta ejecutando usando el siguiente código en terminal:

`lsmod | grep i2c`

El comando va a listar los modulo iniciando con i2c, si en terminal aparece el `“i2c_bmc2708”` quiere decir que todo está correcto.

Es posible realizar un escaneo desde terminal de dispositivos I²C conectados a la Raspberry, para esto tengo un Arduino UNO con I²C conectada a Raspberry.

Para Raspberry modelo B ejecutar el comando:

`sudo i2cdetect –y 1`

Este es el resultado que tengo al conectar un Arduino UNO, me muestra que la dirección es 0x20 (32 en decimal).

``
pi@raspberrypi ~ $ i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: 20 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
``

- Configuración del puerto HOSTPOT

# Implementación

- Diagrama de bloques

- Criterios de diseño

- Esquematico del circuito

- Evidencias

- Código utilizado

Maestro:

Esclavo:



# Conclusiones 

# Referencias


