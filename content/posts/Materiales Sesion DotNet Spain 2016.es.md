---
title: "Materiales de mi sesión en la DotNet Spain 2016"
subtitle: "Async Best Practices"
date: 2016-03-04T11:51:50+02:00
featuredImage: /images/posts/DotNet2016Materials.png #better 1000x500
fontawesome: "true"
categories: 
- Development
- Event
tags:
- csharp
- net framework
- Task
- Parallel
- Async
# draft: true
---

## Materiales de mi sesión

Hola de nuevo,

Durante los últimos días me ha contactado bastante gente haciéndome preguntas sobre [mi sesión](/es/dotnet-spain-conference-2016/) en la pasada DotNet Spain Conference.

Aquí van un par de ellas con sus respuestas, por si son de vuestro interés ;)

### 1 - Sobre la Consola

> “Las demos eran un proyecto de consola, pero qué consola era esa? No es el clásico cmd de Windows, verdad?”

Pues no, desde hace tiempo he sustituido el cmd por la estupenda ConEmu, que permite abrir distintos shells (PowerShell, PuTTY, mintty, GViM, etc) y distribuirlos como tabs o mosaicos.

{{< figure src="/images/posts/ConEmu.png" title="ConEmu" >}}

Podéis descargarla desde: <https://conemu.github.io/>

{{< admonition tip "Nota (Ago. 2020)" >}}
En lugar de ConEmu, como ha llovido algo desde el post original, os recomiendo también el estupendo y renovado [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?activetab=pivot:overviewtab) :D
{{< /admonition >}}

### 2 - Plantilla SQL Profiler

> “En la última demo (la de cancelaciones a la BD) mostraste trazas en el SQL Profiler. Dijiste que tenías una plantilla creada para eso? La podrías compartir si no te importa?”

Por supuesto. Esta plantilla básicamente muestra los eventos Starting y Completed de cada vez que se ejecuta un SQL statement (en este caso un Stored Procedure que tarda unos 5 segundos en completarse). Lo más interesante es que gracias a esta plantilla podemos ver si las peticiones terminan con éxito o han sido canceladas por el usuario, amén de la duración, que también nos puede indicar si ha terminado con éxito o no.

SQL Profiler template: <http://1drv.ms/1p43ti6>

{{< figure src="/images/posts/SqlProfier1.png" title="SQL Profiler template" >}}

Para instalar la plantilla en tu SQL Profiler basta con descargarla y hacer doble click. Se abrirá el profiler y os informarà de que la plantila se ha importado con éxito.

A partir de aquí, cada vez que queráis usar esta plantilla de traza, basta con usarla como las que vienen ‘out of the box’:

{{< figure src="/images/posts/SqlProfier2.png" title="Nueva traza basada en el template" >}}

Un saludo!
