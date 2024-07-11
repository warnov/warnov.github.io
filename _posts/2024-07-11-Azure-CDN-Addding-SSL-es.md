---
title: Cómo Agregar un Certificado SSL Externo a un Endpoint en Azure CDN
date: 2024-07-11 02:53:00 -0500
categories: [Azure, Seguridad y Configuración]
tags: [azure, CDN, SSL, azure key vault, certificates] 
image:
  path: /assets/img/posts/2024-07-11/2024-07-11-Azure-CDN-Addding-SSL-Header.jpg
---
Implementar un certificado SSL para tu endpoint de Azure CDN es crucial para asegurar las comunicaciones y garantizar que el contenido entregado a través de tu CDN, especialmente sitios web estáticos, cumpla con los estándares de seguridad de los navegadores modernos que exigen HTTPS. Esta guía detalla los pasos para agregar un certificado SSL externo a tu endpoint de Azure CDN, enfocándose particularmente en el uso de Azure Key Vault y una Autoridad Certificadora (CA) externa para dominios apex.

## Entendiendo los Certificados SSL para Azure CDN

**Dominios Apex vs. Subdominios**:
- **Dominio Apex**: Este es tu dominio raíz sin ningún prefijo, como `yourdomain.com`.
- **Subdominios**: Estos incluyen cualquier variación como `www.yourdomain.com`, `support.yourdomain.com`, o `blog.yourdomain.com`.

Azure CDN puede gestionar automáticamente certificados SSL para subdominios, pero la configuración manual es necesaria para dominios apex.

## Crear un Certificado en Azure Key Vault

Crear un certificado en Azure Key Vault implica seleccionar el tipo correcto de autoridad certificadora:

1. **Seleccionar el Tipo Correcto de Certificado en Azure Key Vault**:
    - Navega a Azure Key Vault e inicia el proceso para [crear un nuevo certificado](https://learn.microsoft.com/en-us/azure/key-vault/certificates/create-certificate-signing-request?tabs=azure-portal).
    - Elige un certificado emitido por una **CA no integrada**. Esta selección es crucial porque te permite usar Autoridades Certificadoras externas que proporcionan una aceptación y confianza más amplias que las opciones auto-firmadas y podría ser más económico que una CA integrada. También, deja el contenido como `PKCS` ya que es el válido en el CDN.

## Generar y Recuperar el CSR

Crear el CSR en Azure Key Vault y recuperarlo para enviarlo a una CA son pasos clave para obtener tu certificado:

1. **Generar y Descargar el CSR**:
    - En Azure Key Vault, selecciona el certificado que estás creando.
    - Usa las "Operaciones de Certificado" para encontrar y seleccionar "Download CSR". El CSR (Certificate Signing Request) es solo un bloque de texto plano que incluye información codificada sobre tu empresa y el nombre de dominio para el cual se solicita el certificado.

## Enviar el CSR a una CA Externa

Una vez que tienes el CSR, el siguiente paso es tenerlo firmado por tu CA externa elegida.

1. **Enviar el CSR**:
    - Ve al sitio web de tu CA elegida (por ejemplo, Namecheap) e inicia el proceso para obtener un certificado SSL.
    - Necesitarás pegar el CSR en el formulario apropiado en el sitio web de la CA.
    - Especifica el dominio exacto que el certificado debe cubrir (`yourdomain.com`). Esta especificación es crucial porque el certificado SSL estará explícitamente vinculado a los nombres de dominio que especifiques aquí.

2. **Completar la Verificación del Dominio**:
    - La CA típicamente requerirá que demuestres que controlas los dominios listados en tu CSR. Esto puede hacerse a través de la configuración de DNS, verificación por correo electrónico o subiendo un archivo a tu dominio.

## Importar el Certificado Firmado en Azure Key Vault

Después de que tu CA emita el certificado firmado:

1. **Fusionar el Certificado Firmado**:
    - Regresa a Azure Key Vault y ve al mismo certificado.
    - Usa la opción "Merge Signed Request" para subir e integrar el archivo de certificado firmado de la CA con el CSR original.

## Configurar Azure CDN para Usar el Certificado SSL

Configurar tu Azure CDN para usar el nuevo certificado SSL implica algunos pasos adicionales:

1. **Asignar Identidad Gestionada y Permisos**:
    - [Asigna una identidad gestionada por el sistema a tu Azure CDN](https://learn.microsoft.com/en-us/azure/cdn/cdn-custom-ssl?tabs=option-2-enable-https-with-your-own-certificate#select-the-certificate-for-azure-cdn-to-deploy) que le permita acceder de manera segura a Azure Key Vault. Aunque la documentación referencia un procedimiento basado en Key Vaults que trabaja con `Access Policies`, podrías dar permisos para tu CDN al Key Vault usando el enfoque moderno y más recomendado `Access Control IAM`. En este caso, en lugar de elegir permisos, tienes que elegir el rol `Certificate User`.
    - Asigna los permisos de Certificado: Get y List (para Access Policies) **o**, el rol de "Certificate User" (para IAM) a esta identidad dentro de Key Vault para habilitar que el CDN lea el certificado.

2. **Habilitar HTTPS de Dominio Personalizado**:
    - En la configuración de tu endpoint de Azure CDN, selecciona "Custom Domain HTTPS," elige tu Key Vault, y selecciona el certificado.

## Pruebas y Monitoreo

- **Verificar la Funcionalidad HTTPS**: Accede a tu dominio vía HTTPS y verifica que no haya advertencias de seguridad de los navegadores.
- **Monitorear la Expiración del Certificado**: Mantén un registro de cuándo expira tu certificado y planifica su renovación.

## Conclusión

Agregar un certificado SSL externo a un endpoint de Azure CDN asegura que tu contenido se entregue de manera segura, mejorando la confianza del usuario y cumpliendo con los estándares de seguridad. Al seleccionar cuidadosamente el tipo de certificado en Azure Key Vault, generar y enviar un CSR, y configurar Azure CDN, estableces una base segura para la entrega de contenido de tu CDN.
