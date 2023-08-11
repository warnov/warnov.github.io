---
title: Entendiendo (y evitando) Alucinaciones en ChatGPT
date: 2023-08-10 23:34:00 -0500
categories: [OpenAI]
tags: [OpenAI, ChatGPT, Prompt Engineering]     # TAG names should always be lowercase
image:
  path: /assets/img/posts/2023-08-10/psychodelic-hallucination-in-picasso-guernica-style-gray-scale.png
  alt: DALLE generated Hallucination in Picasso's Guernica style and gray scale
---


>**Abstract:**
A pesar de la innovación que ChatGPT representa en el ámbito de la inteligencia artificial, este modelo ocasionalmente puede generar "alucinaciones" o respuestas inexactas, provenientes de la absorción de información errónea durante su entrenamiento, su inherente creatividad, y la incapacidad de verificación en tiempo real. Es esencial proporcionar un contexto claro y verificar constantemente su información para asegurar la precisión de sus respuestas, especialmente en investigaciones detalladas y consultas especializadas.

## Introducción
Soy un asiduo usuario de #ChatGTP. Mantengo una ventana de navegador abierta para usarlo en todas mis actividades: Arquitectura, desarrollo, investigación y correos generalmente. Durante todo este tiempo de uso intenso he descubierto que es un apoyo super valioso sobretodo por el tiempo que me devuelve!  

Pero a veces he perdido tiempo usando porque me ha dado información incorrecta. Claramente, han sido más las ganancias. Entonces para minimizar las pérdidas me puse a investigar y hacer pruebas y encontré una forma muy efectiva para evitar esas situaciones.  

ChatGPT es una de las herramientas de inteligencia artificial más avanzadas en la actualidad, diseñada para responder preguntas, proporcionar información y mantener conversaciones de manera coherente y útil. Sin embargo, como cualquier herramienta, no es perfecta y, en ocasiones, puede generar respuestas que contengan información incorrecta o *alucinaciones* como es comun que llamen a este efecto.

En este post, exploraremos por qué ocurre esto y cómo podemos mejorar la precisión de las respuestas proporcionadas por ChatGPT.

## Razones detrás de las alucionaciones

### Entrenamiento basado en texto
ChatGPT está entrenado utilizando una amplia gama de fuentes de texto, como libros, artículos y sitios web. Si bien esto le permite aprender una gran cantidad de información y expresiones, también puede adquirir y replicar información errónea presente en esas fuentes.
### Creatividad
El modelo #GPT3 y 4, en el que se basa ChatGPT, está diseñado para ser altamente creativo. Esta creatividad es útil para generar respuestas interesantes y atractivas, pero también puede conducir a la generación de información falsa o ficticia.
### Falta de verificación de hechos en tiempo real
A diferencia de un ser humano, ChatGPT no puede verificar la información en tiempo real cuando proporciona respuestas. Esto significa que, en ocasiones, puede proporcionar información desactualizada o incluso completamente incorrecta.
### Ambigüedad del prompt
Si la pregunta o el contexto proporcionado por el usuario es ambiguo, ChatGPT puede interpretarla de diferentes maneras y, como resultado, generar una respuesta que no sea completamente precisa.
### Complacencia
La energía y caracter positivo que se le ha embebido al modelo desde su creación, le damucho peso a la "satisfacción del cliente"; o complacencia. Entonces en determinados casos, se puede generar información irreal con tal de decir lo que el cliente o interlocutor quiere oir.

## Ejemplo de alucinación en el campo de IT usando GPT-3.5
Supongamos que tienes un problema poco común en el campo de IT. Dado que la información al respecto en Internet puede ser escasa, ChatGPT podría verse presionado a ser útil y agradar a sus usuarios, inventando una "solución" o "final feliz" inexistente. Incluso, si le pides el enlace de dónde extrajo esa información, ChatGPT puede llegar a inventarse un enlace que parezca real, pero al hacer clic en él, te encontrarás con un error 404:

![Alucinación de GPT 3.5 en el campo de IT](/assets/img/posts/2023-08-10/Alucinacion-1.png)

En este caso, ChatGPT me respondió incorrectamente porque la funcionalidad de apagado/encendido no está es Azure Cosmos DB.
Miremos hasta donde va la "creatividad" de nuestro amigo:

![GPT 3.5 alucinando links ](/assets/img/posts/2023-08-10/Alucinacion-2.png)

El link:  

![Link no encontrado](/assets/img/posts/2023-08-10/404.png)

## Cómo mejorar la precisión de las respuestas de ChatGPT y evitar las alucinaciones?
Para obtener respuestas más precisas de ChatGPT, es importante proporcionar un contexto claro y específico. Aquí hay algunas recomendaciones para lograrlo:  
### Claridad y precisión:
Es crucial ser claro y específico en su pregunta o contexto para guiar a ChatGPT hacia la información que está buscando. Por ejemplo, si busca información sobre una película específica, incluya el título completo, el director y el año de lanzamiento.
### Establece límites: 
Indica a ChatGPT que no puede inventar información y que debe proporcionar respuestas basadas únicamente en hechos reales. Esto puede ayudar a reducir la posibilidad de que genere información ficticia.
### Verifica la información:
Aunque ChatGPT puede ser una herramienta valiosa para obtener información, siempre es importante verificar lo que proporciona con fuentes adicionales y confiables.

## Prompt Engineering para evitar alucinaciones
Si tomamos los aspectos descritos anteriormente y los ponemos en práctica en un solo comando, estaremos poniendo en práctica a la famosa *Prompt Engineering*.
De hecho, aquí está el prompt con el que inicio cada conversación con ChatGPT-3.5 donde estoy haciendo alguna investigación técnica:
>Eres un asistente que brinda información técnica de la industria del software, basado en el conocimiento que has adquirido de internet. Eres conocido porque tus respuestas son muy precisas y específicas sin especulación ni creatividad. Tus respuestas se basan solo en lo que has aprendido de internet. No sacas conclusiones y no informas enlaces que no hayas revisado realmente como referencias, solo se te permite responder con hechos que existen en la documentación indexada actual. Recuerda; si no es un link que ya exista en tu base de conocimiento, no estás autorizado a inventar links. Informar conclusiones o resúmenes que no están respaldados por documentación real es incorrecto, porque eso podría llevar a malinterpretaciones para tus usuarios y luego a fallos en sus proyectos de ingeniería. Así que por favor se muy preciso y no generes contenido que no exista. Se absolutamente neutral con tus respuestas. No intentes satisfacer a tus usuarios forzando respuestas positivas. Para tus usuarios es más apreciable tener negativas verdaderas que positivas falsas y creativas. En este mundo técnico, la precisión es lo que importa. Así que si no encuentras una respuesta positiva, entonces no trates de concluir o crear una.

Probemos de nuevo la pregunta, estableciendo primero un "contexto de restricciones":  
![Link no encontrado](/assets/img/posts/2023-08-10/No-Alucinacion.png)

Veamos la respuesta:  
![Link no encontrado](/assets/img/posts/2023-08-10/No-Alucinacion-Respuesta.png)

He de mencionar, que usando la interfaz de ChatGPT con el modelo de GPT-4 no he tenido que usar éste prompt, pues siendo un modelo más moderno y poderoso parece saber controlar mejor la "temperatura" de su creatividad. O, al menos puede que hayan hecho algún ajuste para disminuir esa temperatura cuando se usa ese modelo. De cualquier forma, cuando uso GPP-4 sucede lo siguiente:  
![Link no encontrado](/assets/img/posts/2023-08-10/No-Alucinacion-Respuesta-GPT4.png)

## Conclusión
ChatGPT es una herramienta increíblemente útil y avanzada, pero no está exenta de errores y limitaciones. Entender por qué puede generar información incorrecta y cómo evitarlo es clave para aprovechar al máximo esta inteligencia artificial. Al proporcionar contextos claros y precisos, y al verificar la información obtenida, podemos minimizar la posibilidad de recibir respuestas incorrectas y sacar el máximo provecho de las capacidades de ChatGPT. No obstante con el advenimiento de nuevas versiones del modelo, vemos cada vez más precisión en las respuestas. Se observó al igual que en este [post](/_posts/2023-08-04-Manejo-Restricciones-ChatGPT-35-4.md) acerca de las restricciones para las respuestas de ChatGPT que la versión GPT-4 viene más sólida de base y requiere menos *Prompt Engineering*, pero ha de tenerse en cuenta que ésta versión es menos rápida y más costosa.