 Nota: Es necesario tener una placa que soporte modo monitor
 Nota: Cambiar wlp6s0f3u2mon por el nombre de su interfaz en los comandos listados a continuación.
 Nota: Utilizar VM o conectar via ethernet su pc, ya que al cambiar interfaz a modo monitor les va a cortar internet


1. Identificamos la interfaz correspondiente al adaptador utilizando ifconfig, en mi caso **eswlp6s0f3u2** ![Identificando interfaz](Pasted%20image%2020240717151205.png)
2. Con el siguiente comando `sudo airmon-ng start wlp6s0f3u2`, inicializamos el modo monitor en dicha interfaz.
3. Una vez inicializado el modo monitor, podemos comenzar a monitorear las redes wifi a nuestro alcance con el siguiente comando `sudo airodump-ng wlp6s0f3u2mon` (Iniciar el modo monitor agrega el sufijo "mon" a dicha interfaz)  Si queremos incluir redes 5.8 correr el siguiente comando `sudo airodump-ng wlan0 --band abg`
4. Una vez ejecutado dicho comando debemos identificar la red la cual queremos "atacar", para eso anotaremos su "**BSSID**", "**ESSID**" y su canal que corresponde a la columna "**CH**"
5. Una vez obtenida dicha información, procederemos a obtener un dato faltante más que es necesario para realizar el deauth. Para obtenerlo utilizaremos el siguiente comando: `sudo airodump-ng -d BSSID -c CH -w ESSID/Nombre que quieran wlp6s0f3u2mon` y una vez ejecutado anotaremos el valor que figura en la columna "**STATION**" (mac address) (Si el ESSID contiene espacios, escribirlo dentro de comillas dobles **"ESSID"**, de todas formas pueden poner el nombre que quieran, es simplemente el nombre que se le dará al output del proceso) **NOTA: No cortar el proceso del paso 5 hasta luego de haber realizado el paso 6, ya que la idea de airodump-ng es capturar los paquetes de AUTENTICACIÓN que se enviaran desde el dispositivo "ddoseado" hacia la red, para posteriormente intentar crackearlos**.
7. Ya con dicha información procederemos a realizar el envío de paquetes de deautenticación (o desautenticación no se como se dice) con el siguiente comando: `sudo aireplay-ng --deauth 0 -a BSSID -c STATION wlp6s0f3u2mon` (en caso de error modificar el 0 por otro numero mayor a cero) y luego veremos algo similar a esto ![Salida de aireplay-ng](Pasted%20image%2020240717153831.png)
8. Una vez ejecutado esto, encontraremos varios archivos en el directorio donde ejecutamos el comando del paso 5. Uno de ellos es un .cap, que debe contener los paquetes de autenticación de los dispositivos de la red atacada, que intentan conectarse manual o automaticamente.  Algo que no está de más aclarar, volviendo a hacer referencia a los pasos anteriores, una vez que finalizamos la ejecución del deauth, en el proceso del paso 5 deberíamos ver el mensaje WPA handshake: BSSID![WPA handshake capturado](Pasted%20image%2020240717161256.png) Esto nos indica que fue capturado el 4way handshake con los paquetes de autenticación (contraseña de la red) encriptados.
9. Un paso que no es necesario, pero está bueno para entender como funciona es abrir nuestro archivo de airodump con wireshark: `wireshark nombre.cap` y buscamos "eapol" en la barra de busqueda de wireshark y podremos ver estos paquetes mencionados anteriormente ![Paquetes mencionados](Pasted%20image%2020240717161616.png) Si seleccionamos el Message 2 veremos que dentro está el siguiente dato ![Dato dentro del mensaje](Pasted%20image%2020240717161919.png) Esa es la clave de red que debemos crackear que fue capturada por nosotros.
10. Finalmente llegamos a la etapa de crackeo, lo haremos con una wordlist a elección y utilizando como archivo a crackear el mismo que abrimos en wireshark, el .cap. El comando es el siguiente: `sudo aircrack-ng nombre.cap -w wordlist.txt` y si tenemos suerte vamos a recibir el siguiente mensaje en consola ![Crackeo exitoso](Pasted%20image%2020240717162822.png)
------------------------------
Useful Commands:

- arp -a
Associates ip with mac address
- netstat -ano
Shows all open ports and what's connected to those ports
- 

-----------------
Para el cliente:

Si el deauth funciona recomendar lo siguiente.
- Utilizar WPA3
- Si no pueden utilizar WPA3, habilitar Protected Management Frames (PMF), si es soportado por sus routers.
- Monitorear actividad de ret con, por ejemplo, snort
- Configuración de router, (Shorten Beacon Interval, Increase Minimum Signal Strength)
- Educar usuarios, en caso de que sufran desconexiones o no puedan logearse, que avisen inmediatamente

- ### Example: Enabling 802.11w (if supported)

1. **Log in to Your Router's Admin Interface**: Access your router’s web interface, typically via a web browser at an IP address like `192.168.1.1`.
    
2. **Navigate to Wireless Security Settings**: Find the wireless security settings section.
    
3. **Enable 802.11w/PMF**: Look for an option to enable 802.11w or Protected Management Frames (PMF). Enable it.
    
4. **Save and Apply Changes**: Save the settings and reboot your router if necessary.

