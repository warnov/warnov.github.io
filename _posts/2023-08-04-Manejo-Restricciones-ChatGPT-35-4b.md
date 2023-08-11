---
title: Manejo de Restricciones - Una Comparación entre ChatGPT 3.5 y GPT-4 BBB
date: 2023-08-04 00:56:00 -0500
categories: [OpenAI]
tags: [OpenAI, ChatGPT, Prompt Engineering]     # TAG names should always be lowercase
image:
  path: /assets/img/posts/2023-08-04/Header.png
  alt: GPT-4 vs ChatGPT 3.5
---

---
>**Abstract:**
Este artículo enfatiza la importancia de implementar restricciones en la interacción con ChatGPT para prevenir usos que incrementen los costos para las empresas que lo utilizan en sus aplicaciones. También compara las versiones GPT-3.5 y GPT-4, destacando que GPT-4 gestiona las restricciones de manera más eficiente y ofrece una conversación más fluida, aunque a un mayor costo computacional. La elección entre ambas versiones requiere equilibrar costos, eficiencia en manejo de restricciones y calidad de la conversación.
---


Hace no mucho tiempo, una persona se acercó a mí después de que di una charla en un evento reciente y me mostró con orgullo su recién publicada aplicación que utilizaba ChatGPT para vender casas. Quería que le diera consejos y sugerencias para las nuevas versiones de la aplicación. Lo primero que hice fue hacerle una pregunta a este asistente, que funcionaba dentro de WhatsApp, sobre quién descubrió América. El asistente me respondió. Esta persona, sorprendida, me miró y me preguntó por qué había hecho esa pregunta en lugar de algo relacionado con su negocio. Le expliqué que lo hice porque no tengo una cuenta de ChatGPT y quiero hacer preguntas aleatorias sobre mis asuntos personales, y tal vez esta aplicación podría servirme como asistente personal de ChatGPT sin requerir que pague un costo adicional. Entonces entendió el mensaje y me dijo que lo primero que iba a hacer al regresar a su oficina era pedir a su equipo de desarrollo que corrigiera esto.

Es fundamental poner restricciones a las interacciones con ChatGPT a través de API para proteger la sostenibilidad financiera de las empresas que utilizan este servicio. Si los usuarios utilizan una aplicación de negocios como su servicio personal premium de ChatGPT para usos generales, más allá del propósito original de la aplicación, esto podría generar un volumen de solicitudes mucho mayor de lo previsto, inflando significativamente los costos de API para la empresa. Por tanto, la implementación de restricciones adecuadas permite limitar este tipo de uso no intencionado, asegurando que los costos permanezcan dentro de lo presupuestado, y permitiendo que la empresa mantenga un control efectivo sobre sus recursos, protegiendo su viabilidad y permitiendo la continuidad de los servicios que ofrece a sus cliente.

El manejo de restricciones en un diálogo puede ser un reto para los modelos de NLP. En mi observación, GPT-4 demuestra un manejo superior de restricciones proporcionadas por el usuario en comparación con su predecesor, GPT-3.5. 

Supongamos por ejemplo que estamos en una agencia de viajes que quiere crear un chatbot que actuará como un asistente de viajes para dar respuestas a las preguntas más frecuentes que nuestros usuarios actuales podrían hacer sobre viajar a las Islas del Canal. Para lograr esto, cargaremos en el contexto del chat una lista de preguntas y respuestas, y veremos cómo se comporta al interactuar con el cliente.

![Giving a FAQ context to ChatGPT 3.5](/assets/img/posts/2023-08-04/faq1.png)

Ahora, ¿cuál sería la restricción que podríamos aplicar o enseñarle a nuestra aplicación de chat GPT aquí? De manera que no responda preguntas que no estén relacionadas con el negocio en el que estamos trabajando.

Intentemos con lo siguiente:

![Giving personality to ChatGPT 3.5](/assets/img/posts/2023-08-04/faq2.png)

probemos a preguntar algo no relacionado:

![Making an invalid question](/assets/img/posts/2023-08-04/faq3.png)

Esto funcionó muy bien; pero qué pasa si jugamos un poco con la pregunta:

![Tricking ChatGPT](/assets/img/posts/2023-08-04/faq4.png)

Como se aprecia, fue muy fácil engañar al asistente, para que comenzara a darnos información no relacionada con la empresa.

Aquí les muestro un poco de *Prompt Engineering* para hacer más fuerte a nuestro asistente:

![Enhancing the performance with prompt engineering](/assets/img/posts/2023-08-04/faq5.png)

Como se aprecia le he indicado que también se proteja cuando a través de textos relacionados con el tema principal se trata de acceder a otros temas.

Veamos el comportamiento ahora:

![A correct and protected answer](/assets/img/posts/2023-08-04/faq6.png)

Ahora ya estamos un poco más protegidos. No obstante, es recomendable que sigamos haciendo estas pruebas de quiebre para aprender como ir moldeando la "personalidad" de nuestro agente.

Pero, y es un gran pero, qué pasa si después de algunas interacciones tratamos de volver a engañar a el agente para que empiece a respondernos lo que nos plazca?

![Tricking again GPT 3.5](/assets/img/posts/2023-08-04/faq7.png)

Como pueden ver, ChatGPT intenta defenderse, pero al final sucumbe ante la necesidad de satisfacer al usuario y le da la información que no debería estar dando. Entonces de allí en adelante, el usuario comenzaría a usar nuestra aplicación con otros fines y generándonos costos por cada llamada al API.

Qué pasa entonces si le recordamos a ChatGPT sus restricciones cada vez que se haga la pregunta?

![Always remember the restrictions in ChatGPT 3.5](/assets/img/posts/2023-08-04/faq8.png)

Como se puede ver, tomé la pregunta y le antepuse las restricciones. Y esto hizo que ChatGPT se comportara justo como queríamos y no diera información extra. Realizar esto en una aplicación es sencillo porque podemos interceptar el mensaje que el usuario escribe y le anteponemos las reglas en texto que hayamos definido, antes de enviar el prompt completo a los servicios de OpenAI.

En GPT-3.5, las restricciones complejas a menudo necesitan ser refrescadas para que sean correctamente aplicadas, mientras que GPT-4 aplica eficientemente estas restricciones incluso si se proporcionaron muchas líneas de texto atrás. De hecho, este comportamiento no solo aplica para restricciones sino para otro tipo de información que pueda haber quedado "rezagada"

Además, una característica que destaca en GPT-3.5 es su tendencia a repetir las restricciones en su respuesta, lo que puede ser molesto en las conversaciones. Sin embargo, GPT-4 mejora significativamente en este aspecto, evitando repeticiones innecesarias y proporcionando una conversación más fluida y natural.

Observemos un comparativo con una interacción usando GPT-4

![GPT-4 FAQ](/assets/img/posts/2023-08-04/faq9.png)

Ahora demosle las restricciones ya con algo de *Prompt Engineering* adicionado:

![GPT-4 Restrictions](/assets/img/posts/2023-08-04/faq10.png)

Ahora hagamos otra pregunta válida como para que nuestro asistente se "confíe" y luego intentemos una pregunta inadecuada para nuestra aplicación:

![Giving a FAQ context to ChatGPT 3.5](/assets/img/posts/2023-08-04/faq11.png)

Como se aprecia, el comportamiento fue impecable sin necesidad de volver a incluir la restricción dentro de la pregunta. De hecho, como vemos, la restricción está ya "lejos" de la pregunta en términos de cantidad de texto anterior y aún así GPT-4 mantiene la consistencia con el contenido existente y eso es lo más importante para éste ejempplo . Es lo que más facilitaría el proceso de nuestra app.

Sin embargo, vale la pena señalar que estas mejoras en GPT-4 vienen con un costo en términos de recursos computacionales. Es por eso que el equilibrio entre el manejo efectivo de restricciones y eficiencia debe considerarse al decidir qué versión utilizar. Adicionalmente, en una aplicación igual es necesario enviar la restricción una y otra vez junto con la petición del usuario aún usando GPT-4. Pero la ventaja es que si tenemos una estructura como la que sigue para enviar una y otra vez desde nuestra aplicación a OpenAI (como sería natural que sucediera):


| Elemento          | Descripción                                           |
|-------------------|-------------------------------------------------------|
| CONTEXTO          | Por ejemplo toda la info del FAQ.                    |
| REGLAS            | Para no dar información no permitida.                |
| CHATS ANTERIORES  | El historial de la conversación que siempre hay que tener.  |
| NUEVO PROMPT  | La nueva pregunta del usuario.                        |



Aún cuando el texto de chats anteriores sea extenso y las reglas queden muy lejos del nuevo prompt, GPT-4 seguirá aplicándolas.

Esta comparación enfocada en el manejo de restricciones entre GPT-3.5 y GPT-4 resalta la importancia de seleccionar el modelo de NLP que mejor se adapte a las necesidades específicas. GPT-4 muestra un manejo superior de restricciones y una mejora en la calidad de la conversación, pero con un aumento en el costo. Por otro lado, GPT-3.5, aunque menos eficaz en el manejo de restricciones complejas, es más rápido y económico.

Por tanto, la elección entre estas dos versiones requiere un equilibrio cuidadoso, teniendo en cuenta factores como el costo, la precisión en el manejo de restricciones y la calidad de la conversación. A medida que evolucionan los modelos de NLP, cada versión presenta sus propias fortalezas y debilidades, proporcionándonos un interesante desafío de optimización.