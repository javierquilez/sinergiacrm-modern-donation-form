# Formulario Captación Multipaso para SinergiaCRM

Este proyecto proporciona un **frontend multipaso en JavaScript vanilla (sin dependencias)** para captación de fondos con **SinergiaCRM**. Sustituye los webforms nativos del CRM por una única interfaz centrada en la conversión, manteniendo validaciones en cliente y envío **POST** directo al *entryPoint* del CRM.

---

## Flujo del formulario

**Paso 1. Importe y tipo de aportación**  
El usuario elige el importe, define si será puntual o periódica, y visualiza una estimación de desgravación fiscal en tiempo real. Debajo del importe hay un enlace de ayuda que abre la explicación detallada en una ventana modal.

![Paso 1](https://tecnologiasolidaria.org/wp-content/uploads/2025/11/form_donacion_1.png)

**Paso 2. Datos de la persona**  
Se solicitan los datos identificativos y de contacto necesarios para registrar la donación en el CRM y emitir acreditación fiscal.

![Paso 2](https://tecnologiasolidaria.org/wp-content/uploads/2025/11/form_donacion_2_rgpd.png)

**Paso 3. Método de pago**  
Se elige entre **tarjeta** o **domiciliación bancaria (IBAN)**. Si se escoge domiciliación, el formulario pide IBAN (validación básica). Tras confirmar, se genera el POST hacia SinergiaCRM.

![Paso 3](https://tecnologiasolidaria.org/wp-content/uploads/2025/11/form_donacion_3.png)

---

## Configuración necesaria

El formulario trae un bloque explícito de **configuración para producción** en un objeto `formConfig` con cuatro casos de uso. Debes completar **IDs de campaña**, **plantillas de email**, **tipo de relación** y **URLs de redirección** para cada combinación:

```js
// CONFIGURACIÓN PARA PRODUCCIÓN
const formConfig = {
  'persona_donativo': {
    campaign_id: 'ID_DE_TU_CAMPAÑA_PERSONA_DONANTE',
    email_template_id: 'ID_DE_TU_PLANTILLA_PERSONA_DONANTE',
    relation_type: 'donor',
    redirect_url: 'URL_EXITO_DONANTE',
    redirect_ko_url: 'URL_ERROR_DONANTE'
  },
  'persona_cuota': {
    campaign_id: 'ID_DE_TU_CAMPAÑA_PERSONA_SOCIO',
    email_template_id: 'ID_DE_TU_PLANTILLA_PERSONA_SOCIO',
    relation_type: 'member',
    redirect_url: 'URL_EXITO_SOCIO',
    redirect_ko_url: 'URL_ERROR_SOCIO'
  },
  'organizacion_donativo': {
    campaign_id: 'ID_DE_TU_CAMPAÑA_ORG_DONANTE',
    email_template_id: 'ID_DE_TU_PLANTILLA_ORG_DONANTE',
    relation_type: 'donor',
    redirect_url: 'URL_EXITO_ORG_DONANTE',
    redirect_ko_url: 'URL_ERROR_ORG_DONANTE'
  },
  'organizacion_cuota': {
    campaign_id: 'ID_DE_TU_CAMPAÑA_ORG_SOCIO',
    email_template_id: 'ID_DE_TU_PLANTILLA_ORG_SOCIO',
    relation_type: 'member',
    redirect_url: 'URL_EXITO_ORG_SOCIO',
    redirect_ko_url: 'URL_ERROR_ORG_SOCIO'
  }
};
```

Además, el **destino de envío** se define al construir el formulario dinámico. Localiza la línea con `f.action` y sustituye `[INSTANCIA]` por tu subdominio de SinergiaCRM:

```js
f.action = 'https://[INSTANCIA].sinergiacrm.org/index.php?entryPoint=stic_Web_Forms_save';
```

Ejemplo realista:

```js
f.action = 'https://mi-entidad.sinergiacrm.org/index.php?entryPoint=stic_Web_Forms_save';
```

---

## Estilo y branding

El color de marca está implementado en **CSS**, no en configuración JavaScript. En la cabecera (`<style>`) se usa el color **#c21630** para títulos, botones, indicadores de paso y acentos. Si deseas adaptar tu identidad, **reemplaza #c21630** por tu color corporativo en las reglas CSS pertinentes (`h2`, `.btn`, `.selector-group button.active`, `.step-circle.active`, etc.).

---

## Validaciones en cliente

El formulario aplica validaciones prácticas antes del envío:  
• **Documento**: NIF/NIE/CIF con control de formato.  
• **Email**: sintaxis estándar.  
• **IBAN**: si se elige domiciliación, verifica patrón básico **ES** + 22 dígitos (la comprobación definitiva se hace en servidor).  
• **Obligatorios**: compone el parámetro `req_id` con los campos requeridos según el caso y añade valores ocultos para campañas, plantillas y relación.

---

## Gestión de RGPD y Campos del CRM

El Paso 2 implementa la lógica de captación de consentimiento RGPD mediante dos casillas de verificación (checkboxes). Esta lógica se aplica de forma diferenciada si el donante es una "Persona física" o una "Empresa".

1.  **Casilla 1: Política de Privacidad (Obligatoria)**
    * **Comportamiento:** Esta casilla es **obligatoria**. El usuario no puede avanzar al Paso 3 (Pago) si no la marca. La validación se ejecuta al pulsar "Siguiente" y de nuevo en la validación final.
    * **Datos enviados:** Si se marca, el formulario envía los tres campos base de RGPD al CRM:
        * `lawful_basis`: "consent"
        * `date_reviewed`: La fecha actual (formato `YYYY-MM-DD`).
        * `lawful_basis_source`: "website"

2.  **Casilla 2: Comunicaciones de Marketing (Opcional)**
    * **Comportamiento:** Es opcional y no bloquea la donación.
    * **Datos enviados:** Si se marca, envía un `1` a un campo *checkbox* personalizado (ej. `..._stic_gdpr_marketing_c`) que la entidad debe crear.

### Requisitos de Configuración en SinergiaCRM

El formulario está preparado para enviar estos datos, pero SinergiaCRM debe estar configurado para recibirlos correctamente:

* **Para Personas (Módulo `Contacts`):**
    * **Casilla 1 (Privacidad):** Los 3 campos (`lawful_basis`, `date_reviewed`, `lawful_basis_source`) **son estándar** en el módulo `Contacts` y funcionarán sin configuración adicional.
    * **Casilla 2 (Marketing):** **Se debe crear un campo personalizado** (ej. un *checkbox*) para recibir esta aceptación.

* **Para Organizaciones (Módulo `Accounts`):**
    * El módulo `Accounts` no incluye campos RGPD por defecto.
    * Por lo tanto, **la entidad debe crear manualmente los 4 campos** en el módulo `Accounts` para que la integración sea completa (los 3 campos de base legal + el campo de marketing).
---

## Envío al CRM

Al pulsar “FES LA TEVA DONACIÓ →” se construye un formulario **POST** con los campos: importe, método de pago, periodicidad, datos de persona u organización y los metadatos de `formConfig`. Se envía al entryPoint `stic_Web_Forms_save` y se redirige a las URLs de éxito o error configuradas.

---

## Desgravación fiscal en el Paso 1

La cifra que aparece bajo el importe es **orientativa** y se calcula sobre el total **anual** del compromiso (importe × frecuencia). La lógica implementada es:

• **Persona física**: 80 % para los **primeros 250 €**; 40 % para el **exceso** sobre 250 €.  
• **Persona jurídica**: 40 % sobre el **total anual**.

El enlace “Més informació sobre la desgravació” abre una **ventana modal** con el aviso de que es una estimación y puede variar por normativa o ámbito autonómico. La modal se cierra con el icono **✖** en la esquina superior derecha.

---

## Requisitos

Funciona en navegadores modernos con **HTTPS** y acceso desde el sitio público de la entidad al *entryPoint* de SinergiaCRM. No usa librerías externas.

