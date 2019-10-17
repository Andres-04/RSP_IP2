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

Agregue la siguiente información al archivo de configuración. Esta configuración asume que estamos usando el canal 7, con un nombre de red de `Nombre` y una contraseña `Contraseña`. Tenga en cuenta que el nombre y la contraseña no deben tener comillas a su alrededor. La frase de contraseña debe tener entre 8 y 64 caracteres de longitud.

```
interface=wlan0
driver=nl80211
ssid=Nombre
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=Contraseña
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Ahora necesitamos decirle al sistema dónde encontrar este archivo de configuración.

`sudo nano /etc/default/hostapd`

Busque la línea con `#DAEMON_CONF` y reemplácela con esto:

`DAEMON_CONF="/etc/hostapd/hostapd.conf"`

Ahora habilite y comience hostapd:

`sudo systemctl unmask hostapd`
`sudo systemctl enable hostapd`
`sudo systemctl start hostapd`

Haga una comprobación rápida de su estado para asegurarse de que estén activos y en ejecución:

`sudo systemctl status hostapd`
`sudo systemctl status dnsmasq`

Edite `/etc/sysctl.conf` y descomente esta línea:

`net.ipv4.ip_forward=1`

Agregue una mascarada para el tráfico saliente en `eth0`:

`sudo iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE`

Guarde la regla de iptables.

`sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`

Edite `/etc/rc.local` y agregue esto justo arriba de "exit 0" para instalar estas reglas en el arranque.

`iptables-restore < /etc/iptables.ipv4.nat`

Reinicie y asegúrese de que todavía funcione.

Usando un dispositivo inalámbrico, busque redes. El SSID de red que especificó en la configuración de hostapd ahora debería estar presente, y debería ser accesible con la contraseña especificada.



# Implementación

- Diagrama de bloques

- Criterios de diseño

- Esquematico del circuito

Para el esquematico del circuito decidimos utilizar la herramienta Fritzing. Las conexiónes quedaron de la siguiente manera:

![Esquematico_I2C](https://user-images.githubusercontent.com/54821299/67021921-00419680-f0c6-11e9-857d-2cc72cdecfdb.PNG)

- Evidencias

- Código utilizado

Maestro:

```
import smbus #Librerias  para la comunicación
import time
bus = smbus.SMBus(1) # SMBus(1) en RPI 3B y SMBus(0) en RPI A
address = 0x04 #Configuramos la dirección para la comunicación I2C

#Recibimos la temperatura del Arduino UNO y la asignamos a Temp
def Leer(): 
	Temp = bus.read_byte(address)
return Temp

Temp = Leer()
print "La temperatura es: ", Temp #Mostramos la temperatura en pantalla
#Enviamos 1 si Temp>26 y 0 si Temp<=26 para encender o apagar la alarma
def Escribir(value):
	if(Temp > 26):
		Led = bus.write_byte(address, 1)
	if(Temp <= 26):
		Led = bus.write_byte(address, 0)
time.sleep(1)	
```

Esclavo:

```
#include <Wire.h> //Librería para comunicación I2C
#include <DallasTemperature.h> //Librería sensor DS18B20
#define address = 0x04 //Dirección de I2C
int Led = 3; //Led para la alarma
int stateLed = 0; //Estado de la alarma
OneWire ourWire(2); //Definimos el pin del sensor
DallasTemperature sensors(&ourWire);
void setup() {
  delay(1000);
  Serial.begin(9600);
  pinMode(Led, OUTPUT)
  sensors.begin(); // Revisar 
  Wire.begin(address)
  Wire.onReceive(Leer); //Llamamos algunas funciones 
  Wire.onRequest(Escribir);
  Serial.println("Ready!");
}

void loop() {
  sensors.requestTemperatures(); //Avisa que leera la temperatura
  float Temp = sensors.getTempCByIndex(0); //Asigna la temperatura a Temp
  Serial.print("Temperatura= ");
  Serial.print(Temp);
  delay(100);
}
// Si la comunicación con la RSP esta activa le asigna a StateLed el valor recibido
// Si es 1 enciende el led, si es 0 lo apaga
void Leer(int byteCount){
  while(Wire.available()){
    stateLed = Wire.read();
    if(stateLed == 1){
      digitalWrite(Led, HIGH);
    }
    if(stateLed == 0){
      digitalWrite(Led, LOW);
    }
  }
}
//Enviamos el valor de temperatura leido
void Escribir(){
  Wire.write(Temp);
}
```

# Conclusiones 

# Referencias

https://botboss.wordpress.com/2015/07/27/habilitar-comunicacion-i2c-en-rasberry-pi-2/
https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md

