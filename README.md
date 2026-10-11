# 📝 Writeups

Colección personal de *writeups* (resoluciones documentadas) de retos y máquinas de distintas plataformas de ciberseguridad y pentesting: TryHackMe, Hack The Box, OverTheWire, CTFs, etc.

Este repositorio está en constante crecimiento: por ahora solo incluye una plataforma, pero la idea es ir sumando carpetas nuevas a medida que se completen más retos en otros sitios.

> ⚠️ **Uso responsable:** estos writeups son con fines educativos, sobre laboratorios diseñados para practicar (TryHackMe, HTB, CTFs, etc.). No deben usarse como guía para atacar sistemas sin autorización.

## 🗂️ Estructura del repositorio

```
writeups/
├── TryHackMe/      # Writeups de salas de TryHackMe
└── Adjuntos/        # Capturas de pantalla usadas en los writeups
```

Cada plataforma nueva (Hack The Box, OverTheWire, CTFs, etc.) se agregará como una carpeta independiente a este mismo nivel.

## 📚 Writeups disponibles

### TryHackMe

| Room | Tema |
| --- | --- |
| [Billing](https://tryhackme.com/room/billing) | Explotación de un sistema de facturación VoIP y escalada de privilegios |
| [Room 404](https://tryhackme.com/room/hh-room404-804573bf) | Exposición de un repositorio Git en un servidor web |

## 📝 Formato de los writeups

Cada nota sigue, en general, esta estructura:

- Enlace a la sala o reto original.
- Resumen del escenario y de los vectores de ataque usados.
- **Reconocimiento:** herramientas y comandos de enumeración inicial.
- **Explotación:** pasos seguidos para obtener acceso.
- **Escalada de privilegios:** técnica(s) usada(s) para llegar a un usuario con más privilegios.
- Capturas de pantalla de apoyo (carpeta `Adjuntos/`).

Las notas usan enlaces tipo wiki (`[[...]]`) y rutas de imágenes con `![[...]]`, por lo que se leen mejor abriendo el repositorio como bóveda en [Obsidian](https://obsidian.md). En GitHub siguen siendo legibles, pero esos enlaces no se resuelven.

## 🚀 Cómo usarlo

```bash
git clone https://github.com/k3gar/writeups.git
```

## 🧪 Relacionado

Para referencia rápida de los comandos y herramientas usados en estos writeups, ver el repositorio [tools](https://github.com/k3gar/tools).
