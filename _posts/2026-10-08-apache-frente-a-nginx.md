---
layout: post
title: "Apache frente a Nginx"
date: 2026-10-08 17:30:00 +0200
categories: servidores
tags: [apache, nginx, comparativa]
---
Comparamos los dos servidores web más usados en Linux.
| Característica
| Apache
| Nginx
|
|------------------------|-------------------------------|------------------------|
| Modelo
| Procesos / hilos (MPM) | Eventos, asíncrono |
| Configuración
| `.htaccess` por carpeta | Centralizada
|
| Contenido estático | Bueno
| Muy eficiente
|

![IMAGEN](/miblog/assets/images/nginx-apache.png)