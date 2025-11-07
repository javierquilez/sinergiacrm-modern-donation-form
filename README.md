# Formulario Moderno Multipaso para SinergiaCRM

Este proyecto ofrece un frontend moderno, multipaso y 100% personalizable para los formularios de captación de fondos (Web-to-Lead) de **SinergiaCRM**.

Sustituye los formularios web que genera el CRM por una única interfaz que consolida 4 casos de uso en 1, mejorando la experiencia de usuario y aumentando la conversión.

Este formulario está escrito en **JavaScript Vanilla (sin dependencias)**, por lo que se puede integrar en cualquier web (WordPress, Joomla, HTML estático, etc.) simplemente copiando y pegando.

## Problema que Resuelve

Este proyecto ofrece una solución desacoplada que gestiona toda la lógica de validación y configuración en el *frontend* antes de enviar los datos al *backend* de SinergiaCRM.

## Características

* **Formulario Multipaso:** Guía al usuario en 3 pasos (Importe > Datos > Pago).
* **Lógica 4-en-1:** Consolida 4 formularios en una sola interfaz:
    * Persona Física + Donativo (Puntual)
    * Persona Física + Socio (Periódica)
    * Organización + Donativo (Puntual)
    * Organización + Socio (Periódica)
* **Configuración Centralizada:** Un único objeto JavaScript (`formConfig`) para gestionar todos los IDs de campaña, plantillas de email y URLs de redirección.
* **Validación de Formato (España):**
    * Valida el formato de **NIF**, **NIE** y **CIF** en el cliente.
    * Valida el formato de **IBAN**.
    * Valida el formato de **Email**.
* **UX Mejorada:**
    * Validación "en vivo" (`onchange`) para NIF/Email.
    * Calculadora de desgravación fiscal (con modal informativo).
    * Jerarquía visual de botones (primario/secundario).
* **Cero Dependencias:** Solo HTML, CSS y JavaScript.

## Cómo Usarlo

1.  Descarga el archivo `donacio-es.html` (versión en español) o `donacio-ca.html` (versión en catalán).
2.  Subelo por FTP en tu sitio web.
3.  Edita el archivo y ve a la sección `<script>` al final del todo.

### Paso 1: Configurar la Conexión (Tu Instancia)

Busca esta línea (aprox. línea 518) y reemplaza la URL por la de **tu propia instancia** de SinergiaCRM:

```javascript
f.action = 'https://[INSTANCIA].sinergiacrm.org/index.php?entryPoint=stic_Web_Forms_save';
