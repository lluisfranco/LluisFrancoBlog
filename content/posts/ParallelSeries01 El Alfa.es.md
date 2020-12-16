---
title: "Parallel Series 01 - El Alfa"
subtitle: "Empezemos por el principio"
date: 2011-01-11T11:51:50+02:00
featuredImage: /images/posts/parallel01.png
fontawesome: "true"
categories: 
- Development
- Parallel Series
tags:
- csharp
- net framework
- Task
- Parallel
- Async
# draft: true
---

[Ir al índice de la serie](/es/parallelseries00-index)

## Prologo

Cada versión del .NET framework nos sorprende con una serie de novedades, y para cada uno de nosotros hay -al menos- una que nos roba el corazón. A mi me sucedió con la aparición de [Generics](http://msdn.microsoft.com/en-us/library/ms172192.aspx) con el framework 2.0, los [métodos extensores](http://msdn.microsoft.com/en-us/library/bb383977.aspx) y [LINQ](http://msdn.microsoft.com/en-us/library/bb308959.aspx) en las versiones 3.0 y 3.5 y me ha pasado de nuevo con la [Task Parallel Library](http://msdn.microsoft.com/en-us/library/bb308959.aspx) en la versión 4.0.

Pensándolo detenidamente tampoco no es tan extraño, al fin y al cabo siempre me ha gustado la programación asíncrona (algo que si alguien le llega a contar a alguno de mis primeros maestros de tecnología digital, se habría muerto de la risa). Sin embargo, con los años éste que escribe ha llegado -más por tozudez que por talento natural- a conocer un poco los entresijos de la programación asíncrona. Una disciplina en la que tan importante es saber **lo que hay que hacer**, como **lo que no hay que hacer**.

Así pues, cuando llegó a mis manos la primera preview de Visual Studio 2010, una de las primeras cosas que hice fue mirar que demonios era eso de la Task Parallel Library. Primero porque ya hacía un tiempo que había escuchado acerca de la muerte anunciada de la ley de Moore. Y segundo porque cualquier cosa que hiciese más llevadero el trabajo de realizar y depurar aplicaciones multihilo, bienvenido iba a ser.

{{< figure src="/images/posts/tpl-architecture.png" title="Task Parallel Library" >}}

Desde entonces hasta ahora he tenido la suerte de poder dedicar un poco de tiempo a esta maravilla, de modo que me he propuesto crear una serie bastante larga de posts y vídeos sobre el tema. Más que nada porque a mi juicio hay poca documentación (al menos en Español) y algo así no puede quedar relegado al olvido. Y es que en multitud de ocasiones que cuando le explico a alguien el porqué es importante y en que consiste la TPL, acostumbra a decir: “Que guapo! Pues no tenía ni idea…”. Y eso, me mata.

## Nacen las Parallel Series

En los próximos posts voy a ir definiendo el índice de contenidos de la serie, aunque a medida que crezca la serie es más que probable que vaya siendo modificado y ampliado. También veremos algo de historia para ponernos en contexto, aclararemos conceptos base que van a ser necesarios más adelante, e iremos desgranando los apartados principales de la Task Parallel Library.

La idea es empezar desde abajo e ir subiendo de nivel, hasta llegar a los aspectos más complejos, como los problemas de concurrencia o la depuración de este tipo de aplicaciones. En cada apartado prometo poner al menos un ejemplo y si puedo más, porque los humanos -y los developers heredamos de esta clase base- aprendemos mucho mejor los conceptos teóricos si van acompañados de la práctica.

Nos vemos muy pronto ;-)

A continución...

En el [próximo post](/es/parallelseries02-un-poco-de-historia/) repasaremos la historia de la programación paralela y veremos cómo hemos llegado aquí.

[Ir al índice de la serie](/es/parallelseries00-index)
