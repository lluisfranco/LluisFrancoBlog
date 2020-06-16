---
title: "Luces, cámara... action!"
subtitle: "Delegados sin delegados"
date: 2012-02-08T11:51:50+02:00
featuredImage: /images/posts/LightsCameraAction.png
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

## Magia sin delegados

Los delegados de tipo [Action](http://msdn.microsoft.com/en-us/library/018hxwa8.aspx) son una de las pequeñas maravillas traídas a .NET desde la programación funcional. Pueden definirse como un método que tiene un sólo parámetro (en su sobrecarga más simple) y que no devuelve ningún valor. Habitualmente suelen usarse para almacenar referencias a métodos o para pasar un método como parámetro sin tener que declarar explícitamente un delegado. Basta definir el parámetro con la misma firma que se espera recibir y la magia empieza a actuar.

Un detalle importante que podemos ver al observar la firma de Action<T> es que el tipo T es contravariante, de modo que podemos usar este tipo en cualquier otro tipo derivado.  Si quieres saber más sobre [covarianza y contravarianza en Generics](http://msdn.microsoft.com/en-us/library/dd799517.aspx) dale un buen vistazo a [este post](https://www.eiximenis.dev/posts/2010-11-18-c-bsico-covarianza-en-genricos/) del blog del colega Eduard Tomàs.

Veamos un poco de esta magia. Suponiendo que tenemos un método que admite un parámetro de tipo Action<string> podemos llamar al método y pasarle (o más bien inyectarle) el comportamiento deseado, es decir pasarle un método que cumpla con la firma por parámetro:

{{< highlight csharp "linenos=table" >}}
void test()
{
     string msg = "This is the value...";
     doSomethingWithStringValue(enqueueMessage, msg);
     doSomethingWithStringValue(saveToDatabase, msg);
     doSomethingWithStringValue(writeMessageToConsole, msg);
}
 
private void doSomethingWithStringValue(Action<string> actionToDo, string value)
{
     //do several things with this value
     validateMessage(value);
     compressMessage(value);
     //when finishing...
     actionToDo(value);
}
 
private void enqueueMessage(string value)
{
     //do something & enqueue this value
     Queue<string> messages = new Queue<string>();
     messages.Enqueue(value);
}
 
private void saveToDatabase(string value)
{
     //do something & save to db this value
     addLineToUserLog(value);
}
 
private void writeMessageToConsole(string value)
{
     //do something & output this value
     Console.WriteLine(value);
}
{{< / highlight >}}

Por un lado tenemos tres métodos que hacen cosas distintas pero tienen la misma firma (todos esperan un parámetro de tipo string). Y por el otro tenemos un método que tiene un parámetro de tipo Action<string>. Es decir, este parámetro admite como valor cualquier método que tenga la misma firma que hemos declarado. De este modo, podemos invocarlo varias veces y en cada una de ellas de podemos decir que utilice un método distinto para hacer algo distinto. Muy similar a las funciones asíncronas de Javascript o al patrón [Promise](http://wiki.commonjs.org/wiki/Promises/A).

Bonito, eh? Es lo mismo que utilizar delegados pero, uhm… espera! Si, sin usarlos :smile:

## Actions por todos lados

Pues cada vez son más las clases del framework que hacen uso de este tipo de delegados y de su hermano Func, que viene a ser lo mismo pero retornando un valor. Sin ir más lejos, los métodos extensores de LINQ (Select, Where, OrderBy) utilizan Func y casi toda la TPL se basa en el uso de Action, desde los bucles For y ForEach de la clase estática Parallel, hasta la creación explícita de tareas mediante la clase Task.

Por ejemplo, cuando deseamos ejecutar una tarea de forma asíncrona, podemos utilizar el método StartNew de la clase Task.Factory. Este método tiene una sobrecarga en el que acepta un parámetro de tipo Action o Func, y lo mejor de todo es que puede crearse inline (en línea), es decir en el mismo momento en que se realiza la llamada. Veamos unos ejemplos:

Partiendo de un método simple:

{{< highlight csharp "linenos=table" >}}
private void doSomething()
{
    //Pause for 0 to 10 seconds (random)
    Random r = new Random(Guid.NewGuid().GetHashCode());
    Thread.Sleep(r.Next(10000));
}
{{< / highlight >}}

Puesto que es un método que ni recibe parámetros ni devuelve nada podemos llegar a utilizar su sobrecarga más sencilla:

{{< highlight csharp "linenos=table" >}}
Task.Factory.StartNew(doSomething);
{{< / highlight >}}

Otra opción, si el método tuviese un parámetro int para especificar el número de segundos (en lugar de ser aleatorio) podría ser esta:

{{< highlight csharp "linenos=table" >}}
private void doSomething(int seconds)
{
    int mseconds = seconds * 1000
    Thread.Sleep(mseconds);
}
 
Task.Factory.StartNew(() => doSomething(5));
{{< / highlight >}}

Aquí ya vemos algo más curioso. Algo que seguramente hemos observado muchas veces y utilizado antes: [Una expresión lambda](http://msdn.microsoft.com/en-us/library/bb397687.aspx). Esta expresión es también algo tomado de la programación funcional, y puede leerse como: “va hacia”. En la parte izquierda de la expresión se especifican los  parámetros de entrada o variables (si existen, en este caso no), y en la parte derecha la propia expresión. El caso anterior es tan simple que no tiene parámetros y sólo usamos la parte derecha de la expresión para enviar el valor 5 al método.

Al usar una expresión lambda se permite que las instrucciones contenidas en dicha expresión puedan varias líneas, de modo que también podemos llegar a hacer algo como esto:

{{< highlight csharp "linenos=table" >}}
Task.Factory.StartNew(() =>
{
    int x = 5;
    doSomething(x);
    Console.WriteLine("finished!");
});
{{< / highlight >}}

O directamente esto:

{{< highlight csharp "linenos=table" >}}
Task.Factory.StartNew(() =>
{
    int x = 5;
    int mseconds = seconds * 1000
    Thread.Sleep(mseconds);
    Console.WriteLine("finished!");
});
{{< / highlight >}}

En este caso, podemos incluso omitir el método doSomething y usar el código inline directamente en la llamada a StartNew. No obstante, un consejo: No es conveniente abusar de las expresiones inline, de modo que si tenemos más de 5 ó 6 líneas tal vez será más conveniente refactorizar este código para no hacerlo demasiado complejo y respetar los buenos principios de diseño.

## Ahora con parámetros

Hasta ahora al realizar la llamada siempre hemos usado un delegado de tipo Action sin parámetros, de ahí los paréntesis vacíos en la parte izquierda de la expresión lambda. Sin embargo encontraremos multitud de casos en los que debemos pasar parámetros. Sin ir más lejos el método Parallel.For tiene un parámetro de tipo Action al que hay que pasarle un valor de tipo int, lógico por otra parte ya que dentro de un bucle es muy necesario conocer en todo momento el valor de la iteración:

{{< highlight csharp "linenos=table" >}}
Parallel.For(1, 40, (i) =>
{
    serie.Add(i.Fibonacci());
});
{{< / highlight >}}

Observar que no es necesario definir el tipo de datos de la variable i porque el propio compilador es capaz de inferirlo, pero evidentemente también podemos declarar el tipo previo al nombre de la variable, como siempre (int i).

Podemos pasar tantos parámetros como necesite la Action, el mismo método tiene otra sobrecarga que admite un objeto ParallelLoopState para poder cancelar el bucle:

{{< highlight csharp "linenos=table" >}}
Parallel.For(1, 40, (i, loopState) =>
{
    serie.Add(i.Fibonacci());
    if (i > 35) loopState.Break();
});
{{< / highlight >}}

Y por supuesto podemos crearnos nuestras propias acciones con tantos parámetros como sean necesarios. Aunque al igual que ante, si necesitamos pasar más de 3 ó 4 parámetros a un Action tal vez deberíamos plantearnos si estamos haciendo las cosas bien

{{< highlight csharp "linenos=table" >}}
private void saveToDatabase(string value, bool useDetails)
{
    addLineToUserLog(value);
    if (useDetails) addLineToUserLogDetails();
}

void test()
{
    //Define una acción que apunta al método saveToDatabase
    Action<string, bool> myAction = (v, s) =>
    {
        saveToDatabase(v, s);
    };
 
    string value = "This is the value...";
    bool usedetails = true;
    myAction(value, usedetails); //Aquí se llama a la acción y al método al que apunta
}
{{< / highlight >}}

## Resumiendo

Los delegados de tipo Action son muy útiles para simplificar el trabajo con delegados (ahora que lo pienso hace bastante tiempo que no los uso, ni para declarar eventos). Nos permiten especificar las acciones a realizar pudiendo llegar a tener hasta 16 parámetros -demasiados en mi opinión- y al igual que los método void no devuelven ningún valor. Si queremos lo mismo pero pudiendo retornar un resultado debemos utilizar su hermano Func<T, TResult> que es exactamente igual, pero en todas sus sobrecargas (y tiene tantas como Action) el último argumento representa el valor de retorno.

[Ir al índice de la serie](/es/parallelseries00-index)