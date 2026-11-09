# 🚀 Post para LinkedIn (borrador único — todas las fases)

> Lenguaje: técnico-casual narrativo. Edita a gusto antes de publicar.
> Sugerencia: usar 1 imagen de gancho (la del nmap o la del dashboard) + carrusel con las capturas por fase.

---

🧊 **ColddBox Easy: de cero a ROOT en una tarde (y todo lo que aprendí en el camino)**

Decidí montar un laboratorio con máquinas virtuales para practicar hacking ético de forma 100% segura: una Kali como atacante y una máquina vulnerable llamada ColddBox, todo en una red aislada que controlo yo. Regla de oro: solo se "ataca" lo propio, y el objetivo siempre es aprender a **defender** mejor.

Esto fue lo que pasó, fase por fase 👇

**1 🔍 Reconocimiento**
Todo pentest empieza sabiendo qué hay en el territorio. Con `netdiscover` mapeé la red y encontré a la víctima; un `ping` confirmó que estaba viva.
No es lo más glamoroso, pero el 80% del trabajo es escuchar y observar.

**2 🛰️ Escaneo**
`nmap -p-` para barrer los 65535 puertos: el 80 (web) y el 4512 (SSH en un puerto raro). Con `-sCV` saqué versiones: Apache 2.4.18 y **WordPress 4.1.31**. `whatweb` remató el fingerprint: xmlrpc.php habilitado y tema sin actualizar.
👉 Lección: las versiones desactualizadas son un cartel de "vengo con deudas técnicas".

**3 👤 Enumeración**
El login de WordPress cometía un error clásico: decía "usuario inválido" cuando el usuario no existía. Eso te permite **confirmar cuentas** sin hacer ruidos raros. `wpscan --enumerate u` me dio 4 usuarios en segundos. Objetivo: `c0ldd`. ✅

**4 🔑 Fuerza bruta**
Con el usuario confirmado y rockyou.txt como diccionario, `wpscan` encontró la contraseña en un rato: `9876543210` 🙃 sí, literalmente la secuencia de números.
👉 Lección para defensa: 2FA, limitar intentos y mensajes de error genéricos. Una contraseña débil anula todo lo demás.

**5 🐚 Acceso y shell**
Dentro del panel de WordPress, la joya de la corona: el **editor de temas**. Pegué un reverse shell de pentestmonkey en el `404.php` del tema, puse mi listener `nc -lvnp 9001`, pedí una página inexistente... y obtuve un shell como **www-data**, el usuario del servidor web. Basta con que un solo rol tenga permiso de editar código para que el server sea del atacante.

**6 ⬆️ Escalada a root**
Movimiento lateral al usuario real del sistema con credenciales encontradas durante la enumeración interna. Y entonces, `sudo -l`: el usuario podía ejecutar `vim`, `chmod` y `ftp` como root. Con `chmod` el camino es corto: SUID en `/bin/bash`, `bash -p` y... `whoami` → **root** 👑. La flag final hasta venía con mensaje de felicitación en base64. 🏁

**🎯 Lo que me llevo de este laboratorio:**
✅ Un pipeline completo mental: recon → enum → exploit → shell → privesc → flag
✅ La cadena de ataque más peligrosa casi siempre es una _suma_ de detalles pequeños (errores de login + contraseña débil + permisos mal asignados)
✅ Defender es entender el ataque: cada hallazgo aquí es una recomendación concreta (bloquear el editor de temas, restringir sudo, monitorizar wpscan...)

⚠️ Todo se hizo en un entorno propio y controlado, con fines educativos. El hacking ético es estar autorizado, documentar y ayudar a cerrar puertas, no abrirlas.

¿Qué máquina me recomiendan para la siguiente práctica? 👇

#CiberSeguridad #HackingÉtico #Pentesting #Linux #WordPress #CyberSecurity #Aprendizaje Continuo
