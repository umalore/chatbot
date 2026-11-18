# 📅 Asistente Virtual de Agendamiento de Turnos (IA)

Aplicación web en producción que permite a los usuarios consultar disponibilidad horaria, agendar nuevas citas y cancelar turnos existentes mediante lenguaje natural, sincronizando automáticamente una base de datos relacional y Google Calendar.

---

## 🌐 Enlace en Producción
* **Aplicación Web (Vercel):** `[PEGAR AQUÍ TU URL DE VERCEL]`

---

## 🏗️ Arquitectura del Sistema

El proyecto utiliza una arquitectura desacoplada orientada a eventos:

1. **Frontend (HTML5, CSS3, JavaScript Vanilla — Vercel):**
   * Interfaz de chat responsiva accesible desde computadoras y dispositivos móviles.
   * Envía las consultas del usuario mediante peticiones asíncronas `POST` (`fetch`) en formato JSON hacia un Webhook seguro y renderiza la respuesta en tiempo real.
2. **Orquestador de Flujos (Make.com):**
   * Recibe la petición HTTP a través de un **Custom Webhook** activo 24/7[cite: 5].
   * Utiliza un módulo **Parse JSON** con estructura de datos tipada para validar los campos extraídos por la IA antes de enrutarlos[cite: 5].
   * Implementa un **Router** con tres ramas operativas[cite: 5]:
     * **Rama 1 (`buscar`):** Consulta eventos ocupados en Google Calendar aplicando validación de fecha por defecto (`ifempty(now)`) y calcula los horarios disponibles entre las 09:00 y las 18:00 hs.
     * **Rama 2 (`agendar` — Fallback):** Crea el evento en Google Calendar, registra al cliente y la cita en Supabase vinculando el `event_id`, y confirma la reserva[cite: 5].
     * **Rama 3 (`cancelar`):** Busca la cita en Supabase, elimina el evento correspondiente en Google Calendar mediante su `event_id` y actualiza el registro en la base de datos[cite: 5].
3. **Procesamiento de Lenguaje Natural (Google Gemini AI):**
   * **Clasificación y Extracción:** Analiza la intención del usuario con reglas de jerarquía estricta y calcula fechas relativas (ej. *"hoy"*, *"mañana"*) a partir de la fecha y hora actual del servidor (`formatDate(now)`).
   * **Generación de Respuestas:** Redacta respuestas naturales y estructuradas en JSON plano.
4. **Base de Datos (Supabase — PostgreSQL):**
   * Persistencia de clientes y reservas históricas.
5. **Gestión de Agenda (Google Calendar API):**
   * Sincronización bidireccional en tiempo real para evitar superposición de turnos[cite: 1, 5].

---

## 🗄️ Diseño de la Base de Datos (Supabase)

La base de datos está normalizada en dos tablas principales:

| Tabla | Columnas Clave | Descripción |
| :--- | :--- | :--- |
| **`usuarios`** | `id`, `nombre`, `apellido`, `telefono` | Almacena la información de contacto de los pacientes/clientes que solicitan turnos. |
| **`turnos`** | `id`, `fecha`, `hora`, `servicio`, `estado`, `event_id` | Registra cada cita agendada. La columna `event_id` guarda el identificador único generado por Google Calendar para permitir futuras cancelaciones exactas. |

---

## 🔒 Seguridad y Buenas Prácticas

* **Protección de Credenciales:** Todas las llaves de API y tokens de acceso (Google Gemini, Google Calendar y Supabase) residen exclusivamente dentro de las conexiones cifradas de Make.com. Ninguna credencial sensible está expuesta en el código fuente del frontend (`app.js`).
* **Sanitización y Manejo de Errores:**
  * Uso de fórmulas defensivas (`ifempty`, `parseDate`) para evitar fallos (`DataError`) cuando el usuario omite fechas explícitas.
  * Prompts con restricción de salida (`JSON crudo sin bloques Markdown`) y jerarquía de prioridades para evitar clasificaciones ambiguas.
* **Control de Versiones:** Código fuente gestionado en GitHub con despliegue continuo automático hacia Vercel.

---

## 🚀 Cómo Funciona (Flujo de Uso)

1. **Consultar disponibilidad:** El usuario escribe *"¿Qué turnos hay libres mañana?"*. El sistema consulta Google Calendar y devuelve las franjas horarias disponibles.
2. **Agendar un turno:** El usuario escribe *"Quiero agendar para el 19 de noviembre a las 11:00, soy Juan Pérez, tel 123456789"*. La IA detecta los datos personales con máxima prioridad, crea el evento en Calendar y guarda la fila en Supabase con su `event_id`.
3. **Cancelar un turno:** El usuario escribe *"Quiero cancelar mi turno del 19 de noviembre a las 11:00, soy Juan Pérez"*. El flujo recupera el `event_id` en Supabase, borra el evento de Calendar y confirma
