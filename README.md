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

```
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
```

- Configuración del puerto HOSTPOT

La Raspberry Pi se puede usar como un punto de acceso inalámbrico, ejecutando una red independiente. Esto se puede hacer usando las funciones inalámbricas incorporadas de Raspberry Pi 3 o Raspberry Pi Zero W, o usando un dongle inalámbrico USB adecuado que admita puntos de acceso.

Para crear un punto de acceso, necesitaremos DNSMasq y HostAPD. Instale todo el software requerido de una vez con este comando:

```sudo apt install dnsmasq hostapd```

Como los archivos de configuración aún no están listos, apague el nuevo software de la siguiente manera:

```
sudo systemctl stop dnsmasq
sudo systemctl stop hostapd
```

Esta documentación asume que estamos utilizando las direcciones IP estándar `192.168.xx` para nuestra red inalámbrica, por lo que asignaremos al servidor la dirección IP `192.168.4.1.` También se supone que el dispositivo inalámbrico que se está utilizando es `wlan0`.

Para configurar la dirección IP estática, edite el archivo de configuración dhcpcd con:

```sudo nano /etc/dhcpcd.conf```

Vaya al final del archivo y edítelo para que tenga el siguiente aspecto:

```
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```

Ahora reinicie el `daemon dhcpcd` y configure la nueva `wlan0` configuración:

```
sudo service dhcpcd restart
```

El servicio DHCP es proporcionado por dnsmasq. De manera predeterminada, el archivo de configuración contiene mucha información que no es necesaria y es más fácil comenzar desde cero. Cambie el nombre de este archivo de configuración y edite uno nuevo:

`sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig`
`sudo nano /etc/dnsmasq.conf`

Escriba o copie la siguiente información en el archivo de configuración dnsmasq y guárdela:

```
interface=wlan0      # Use the require wireless interface - usually wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

Entonces wlan0, vamos a proporcionar direcciones IP entre 192.168.4.2 y 192.168.4.20, con un tiempo de arrendamiento de 24 horas. Vuelva dnsmasqa cargar para usar la configuración actualizada:

`sudo systemctl reload dnsmasq`

Debe editar el archivo de configuración de hostapd, ubicado en /etc/hostapd/hostapd.conf, para agregar los diversos parámetros para su red inalámbrica. Después de la instalación inicial, este será un archivo nuevo / vacío.

`sudo nano /etc/hostapd/hostapd.conf`


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

https://botboss.wordpress.com/2015/07/27/habilitar-comunicacion-i2c-en-rasberry-pi-2/

