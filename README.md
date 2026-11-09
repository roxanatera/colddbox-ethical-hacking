# 🧊 ColddBox Easy — Writeup de Hacking Ético

Laboratorio práctico de **hacking ético** sobre la máquina vulnerables **ColddBox Easy**, ejecutado en un entorno controlado y aislado (VirtualBox), con fines exclusivamente **educativos**.

Máquina atacante: **Kali Linux 2026.3** · Víctima: **ColddBox Easy** (192.168.1.141) · Entorno: VirtualBox en red puente aislada.

---

## ⚠️ Disclaimer

> Todo lo documentado aquí se realizó sobre un **laboratorio propio y controlado** (máquinas virtuales propias en red aislada).
> El hacking ético se practica solo con **autorización explícita**. Aprender estas técnicas sirve para
> **defender** sistemas, no para atacarlos. No uses este conocimiento contra sistemas de terceros.

---

## 🛠️ Entorno utilizado

| Componente | Detalle |
|---|---|
| Máquina atacante | Kali Linux 2026.3 (VirtualBox) |
| Máquina víctima | ColddBox Easy (VirtualBox) |
| Red | Adaptador puente, red local 192.168.1.0/24 |
| IP víctima | 192.168.1.141 |
| Documentación | 28 capturas del proceso completo |

---

## 🗺️ Cadena de ataque (resumen)

```
netdiscover → nmap (puertos 80 y 4512) → whatweb (WordPress 4.1.31)
→ wpscan (enumeración de usuarios) → brute force con rockyou.txt
→ acceso admin WordPress → reverse shell en 404.php del theme
→ shell www-data → TTY upgrade → su c0ldd → sudo chmod (bash SUID) → ROOT 🏁
```

---

# 📖 Writeup paso a paso

## Fase 1 — Entorno y descubrimiento de hosts

**Paso 1 · Preparar el laboratorio**
Dos VMs corriendo en VirtualBox: la víctima (ColddBoxEasy_IV) y Kali como atacante, ambas en adaptador puente.

![Entorno VirtualBox](img/01-entorno-virtualbox.png)

**Paso 2 · Descubrir hosts en la red**
`netdiscover` (captura ARP) revela 13 equipos. Dominio completo del mapa de red. La víctima queda identificada por su MAC de VirtualBox.

![netdiscover](img/02-netdiscover-hosts.png)

**Paso 3 · Confirmar conectividad**
Ping a la víctima: 3/3 paquetes, 0% pérdida.

```bash
ping 192.168.1.141
```

![ping](img/03-ping-conectividad.png)

---

## Fase 2 — Escaneo y reconocimiento

**Paso 4 · Escaneo de todos los puertos**
Escaneo rápido de los 65535 puertos TCP, solo mostrando abiertos.

```bash
sudo nmap -p- --open -T4 192.168.1.141
```

Resultado: **80/tcp** (http) y **4512/tcp** (servicio por identificar).

![nmap all ports](img/05-nmap-allports.png)

**Paso 5 · Detección de versiones**
Escaneo con scripts por defecto y detección de versión sobre los puertos hallados.

```bash
sudo nmap -p80,4512 -sCV 192.168.1.141 -oN escaneo.txt
```

- 80/tcp → **Apache httpd 2.4.18 (Ubuntu)** + **WordPress 4.1.31** (título: *ColddBox | One more machine*)
- 4512/tcp → **OpenSSH 7.2p2** (puerto SSH en ubicación poco común 👀)

![nmap versiones](img/04-nmap-versiones.png)

**Paso 6 · Fingerprinting web**
`whatweb` confirma CMS WordPress 4.1.31, jQuery 1.11.1 y **xmlrpc.php habilitado** (x-pingback) — dato valioso para más adelante.

```bash
whatweb 192.168.1.141
```

![whatweb](img/06-whatweb-fingerprint.png)

**Paso 7 · Reconocimiento visual**
Visita al sitio en el navegador: blog WordPress con un único post de bienvenida.

![Web](img/07-web-wordpress.png)

---

## Fase 3 — Enumeración de WordPress

**Paso 8 · Sondeo manual del login**
Prueba en `wp-login.php` con un usuario inventado ("root"): el mensaje *"ERROR: Invalid username"* revela que la app **distingue usuarios válidos de inválidos** — error clásico de UX que filtra información.

![login inválido](img/08-wp-login-invalido.png)

**Paso 9 · Enumeración automática de usuarios**
```bash
wpscan --url 192.168.1.141 --enumerate u
```

![wpscan](img/09-wpscan-enum.png)

**Paso 10 · Usuarios identificados**
wpscan localiza **4 usuarios**: *the cold in person*, **c0ldd**, *hugo*, *philip*. Además detecta el theme **twentyfifteen v1.0** (desactualizado).

![usuarios](img/10-wpscan-usuarios.png)

**Paso 11 · Confirmar usuario objetivo**
Login manual con `c0ldd`: ahora el error cambia a *"The password you entered... is incorrect"* → el usuario **existe**. Objetivo confirmado ✅

![login válido](img/11-wp-login-valido.png)

---

## Fase 4 — Fuerza bruta de credenciales

**Paso 12 · Preparar el diccionario**
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

![rockyou](img/12-rockyou-descompactar.png)

**Paso 13 · Ataque dirigido**
Solo contra el usuario confirmado, con rockyou como diccionario:

```bash
wpscan --url 192.168.1.141 --usernames c0ldd --passwords /usr/share/wordlists/rockyou.txt
```

![brute force](img/13-wpscan-bruteforce.png)

**Paso 14 · Ataque en curso**
wpscan prueba las contraseñas una a una contra el formulario de login.

![búsqueda](img/14-wpscan-busqueda.png)

**Paso 15 · Credenciales encontradas 🎉**
```
[SUCCESS] - c0ldd / 9876543210
```
Contraseña débil + usuario filtrado = acceso garantizado. Lección para defensas: **limitar intentos de login y cuidar los mensajes de error**.

![success](img/15-credenciales-encontradas.png)

**Paso 16 · Login manual**
Probar las credenciales desde el navegador.

![login manual](img/16-wp-login-manual.png)

**Paso 17 · Acceso al Dashboard ✅**
Sesión iniciada como editor. Desde aquí el servidor es, a efectos prácticos, nuestro.

![dashboard](img/17-wp-dashboard.png)

---

## Fase 5 — Reverse shell

**Paso 18 · Elegir el payload**
Generador online de reverse shells (revshells.com) con listener `nc -lvnp 9001`.

![revshells](img/18-revshells-generador.png)

**Paso 19 · Payload PHP PentestMonkey**
Reverse shell clásico en PHP: conecta de vuelta a la IP del atacante en el puerto 9001.

![payload](img/19-payload-php-pentestmonkey.png)

**Paso 20 · Inyección vía editor de temas**
Centro de la técnica: WordPress permite **editar el código PHP del tema activo** desde el panel de administración. Pegamos el shell en **`404.php`** del theme *Twenty Fifteen*. Cualquier petición a una página inexistente ejecutará el código.

![404.php con shell](img/20-shell-en-404php.png)

**Paso 21 · Listener preparado**
```bash
sudo nc -lvnp 9001
```

![listener](img/21-nc-listener-9001.png)

**Paso 22 · Shell recibido 🔥**
Se dispara el 404 (petición a una página inexistente) y el listener captura la conexión:
`whoami` → **www-data** (uid 33), el usuario del servidor web. Explorando `/var/www/html` aparece un directorio muy prometedor: **`hidden/`** y todo el código WordPress, incluido **`wp-config.php`**.

![shell www-data](img/22-shell-wwwdata.png)
![enumeración](img/23-enumeracion-varwww.png)

**Paso 23 · TTY upgrade**
El shell es estable pero feo. Lo hacemos interactivo:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

→ `www-data@ColddBox-Easy:/var/www/html$`

![tty upgrade](img/24-tty-upgrade.png)

---

## Fase 6 — Escalada de privilegios

**Paso 24 · Movimiento lateral a c0ldd**
Con las credenciales recuperadas durante la enumeración interna (archivos de configuración/datos del sistema), cambiamos de usuario con `su`:

```bash
su c0ldd     # password: **********
```

→ `c0ldd@ColddBox-Easy:/var/www/html$`

![su c0ldd](img/25-su-c0ldd.png)

**Paso 25 · Enumeración de sudo**
```bash
sudo -l
```

c0ldd puede ejecutar **como root**: `/usr/bin/vim`, **`/bin/chmod`** y `/usr/bin/ftp`.
Con `chmod` permitido en sudo la escalada es inmediata:

```bash
sudo /bin/chmod u+s /bin/bash   # SUID en bash
bash -p                          # shell con privilegios del dueño (root)
```

→ `bash-4.3#`

![sudo -l y escalada](img/26-sudo-l-escalada.png)

**Paso 26 · ROOT 👑**
```bash
whoami
→ root
```

![root](img/27-root-confirmado.png)

**Paso 27 · Flag final 🏁**
```bash
cd /root && cat root.txt
→ wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxlZGFkYSE=
```
Base64 → **"¡Felicidades, máquina completada!"** 

![flag](img/28-flag-root-txt.png)

---

## 📌 Lecciones y defensas

| Hallazgo | Defensa recomendada |
|---|---|
| Mensajes de error de login revelan usuarios válidos | Mensaje genérico "usuario o contraseña incorrectos" |
| Contraseña en rockyou para la admin del blog | Políticas de contraseñas + 2FA en `/wp-login.php` |
| xmlrpc.php expuesto si no se usa | Deshabilitarlo o filtrarlo (WAF) |
| WordPress 4.1.31 / theme sin actualizar | Actualización continua de core y temas |
| Editor de temas disponible para el rol editor | Restringir `DISALLOW_FILE_EDIT` / permisos por rol |
| Binarios con sudo sin restricciones (chmod, vim, ftp) | Principio de menor privilegio; `sudo` por comando específico con argumentos acotados |

---

## 🎯 Objetivos pedagógicos

- Pipeline completo de un pentest: **recon → enum → exploit → shell → privesc → flag**
- Enumeración de usuarios en WordPress con wpscan
- Fuerza bruta de credenciales con diccionarios (rockyou)
- Inyección de reverse shells vía editor de temas de WordPress
- Escalada de privilegios con binarios permitidos en sudo (vim, chmod, ftp)
- Documentación y comunicación del proceso (writeup + post)

---

*Realizado por Julia como ejercicio de aprendizaje en ciberseguridad ofensiva. Con fines educativos.* 🚀
