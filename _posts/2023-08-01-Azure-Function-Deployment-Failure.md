---
title: Error inexplicable en el despliegue de una Azure Function
date: 2023-06-30 01:24:00 -0500
categories: [Azure, Functions, Deployment, error]
tags: [azure, azure functions, deployment, error, visual studio]     # TAG names should always be lowercase
image:
  path: /assets/img/posts/2023-06-30-AAC-KV-FX-Header.png
  alt: Azure Function deployment from Visual Studio error dialog box
---

Dos preciosas horas de mi vida desaparecieron gracias a éste críptico mensaje de error que me apareció cuando muy contento yo ya estaba intentado desplegar mi Azure Function en Azure luego de que ya funcionaba perfectamente en mi entorno usando lo último del modelo `Isolated` en `dotnet 7` (aclaro que ésto no tuvo que ver con el error) junto con el excelente servicio para manejo de configuración de aplicaciones `Azure App Configuration` e incluso `KeyVault`! (Puedes consultar más detalles de cómo lograr esto en [éste post](https://blog.warnov.com/posts/AAC-KV-FX/) que escribí al respecto).

Como se nota, no es que el mensaje de error proporcione mucha información. Excepto que se puede ir a ver un archivo de log. Que si lo abrimos nos muestra majestuosamente:

![Archivo de log inútil](/assets/img/posts/2023-08-01-Azure-Function-Deployment-Failure-Useless-Log-File.png)

Lo mismo: nada. Ni una pista de lo que sucede.

Reinicié Visual Studio, la máquina, intenté desde otro PC, luego desde una VM en Azure, y siempre tuve el mismo error, aún cuando ya había podido desplegar esta función antes. Además el hecho de que la function corriera impecablemente en cualquiera de las máquinas que probé hacía que me confundiera más.

Solo fue cuando la divina providencia de pura casualidad hizo que `Visual Studio` me mostrara el `Solution Explorer` pero en modo carpetas `(Folder View)`, que noté algo muy raro:

![Ventajas de la vista de carpeta o Folder View](/assets/img/posts/2023-08-01-Azure-Function-Deployment-Failure-Blessed-Folder-View.png)

Esto me hizo notar que el proyecto de alguna manera se me había duplicado (tal vez un `drag-and-drop`accidental). Y, aunque el proyecto interno era el que tenía la última versión y allí funcionaba correctamente de manera local, al tratar de desplegarlo, el `wizard` de `Visual Studio` se confunde porque se encuentra que el proyecto que intenta desplegar está contenido en otro proyecto. Sin embargo, el hecho de que la function compilará y funcionara bien hacía muy difícil pensar en esto que de casualidad noté.

Cuando noté esto, aún no había empezado a intentar la solución de desplegar por FTP por ejemplo. Aunque, en este caso creo que no habría habido problema porque en el despliegue manual yo solo hubiese subido el contenido de la carpeta `bin`que es la que se requiere para correr la función en Azure. Además viendo la estructura de carpetas en el cliente FTP, muy seguramente habría notado esta irregularidad.

Pues bien, procedí a eliminar los archivos externos y luego moví los internos un nivel más arriba y finalmente eliminé la carpeta que quedó vacía y de esta manera me quedó una estructura de este tipo:

![Estructura de archivos corregida](/assets/img/posts/2023-08-01-Azure-Function-Deployment-Failure-Fixed-File-Structure.png)

Después de esto ya el despliegue funcionó perfectamente!

### Moraleja: 

Si tu aplicación funciona correctamente y estás seguro que todo se ha configurado correctamente para el despliegue y sin embargo sigues teniendo problemas de despliegue, es muy probable que de alguna manera tengas una estructura de archivos errada como en el caso mostrado, que necesite ser corregida.

