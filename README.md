# 🧊 ColddBox Easy — Writeup de Hacking Ético

Laboratorio práctico de **hacking ético** sobre la máquina vulnerables **ColddBox Easy**, ejecutado en un entorno controlado y aislado (VirtualBox), con fines exclusivamente **educativos**.

![Entorno](img/01-entorno-virtualbox.png)

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

---

## 🗺️ Cadena de ataque (resumen)

```
netdiscover → nmap (puertos 80 y 4512) → whatweb (WordPress 4.1.31)
→ wpscan (enumeración de usuarios) → brute force con rockyou.txt
→ acceso admin WordPress → reverse shell en 404.php del theme
→ shell www-data → TTY upgrade → su c0ldd → sudo chmod (bash SUID) → ROOT 🏁
```

---

## 📚 Estructura del repositorio

```
├── README.md            ← esta introducción
├── writeup.md           ← el paso a paso completo con capturas
├── post-linkedin.md     ← borrador del post para LinkedIn
└── img/                 ← 28 capturas del proceso (renombradas por fase)
```

---

## 🎯 Objetivos pedagógicos

- Pipeline completo de un pentest: **recon → enum → exploit → shell → privesc → flag**
- Enumeración de usuarios en WordPress con wpscan
- Fuerza bruta de credenciales con diccionarios (rockyou)
- Inyección de reverse shells vía editor de temas de WordPress
- Escalada de privilegios con binarios permitidos en sudo (vim, chmod, ftp)
- Documentación y comunicación del proceso (writeup + post)

---

## 📖 Cómo usar este repo

1. Lee [`writeup.md`](writeup.md) para ver el paso a paso con cada captura.
2. Consulta [`post-linkedin.md`](post-linkedin.md) para la versión resumida y narrativa.
3. Si quieres replicar el laboratorio: monta la VM en VirtualBox en red aislada, ajusta IPs y sigue el orden de las fases.

---

*Realizado por Julia como ejercicio de aprendizaje en ciberseguridad ofensiva.* 🚀
