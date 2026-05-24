# Jandrea - Chatbot Comercial WhatsApp

## Descripcion General

Chatbot de atencion comercial por WhatsApp para **Jandrea**, una tienda artesanal de manualidades, recuerdos y personalizaciones ubicada en Tumbaco, Quito, Ecuador. Se especializan en corte laser e impresion 3D para crear piezas unicas.

El sistema esta construido sobre **n8n** usando nodos **AI Agent**. Tiene un agente principal (coordinador) y sub-agentes especializados que se activan mediante herramientas (tools).

---

## Modelo de IA

- **Modelo configurado en todos los nodos:** GLM 5.1
- Archivo: `model.txt`

---

## Estructura de Archivos

```
chat-bot-jandrea/
  model.txt                              # Modelo de IA configurado (GLM 5.1)
  system-prompt.txt                      # Prompt del agente principal (Jandrea)
  subagent-mostrar_productos-prompt.txt   # Prompt del sub-agente de productos
  subagent-mostrar_productos-descripcion.txt  # Descripcion/cuando usar el sub-agente de productos
  subagent-mostrar_catalogos-prompt.txt   # Prompt del sub-agente de catalogos
  subagent-mostrar_catalogos-descripcion.txt  # Descripcion/cuando usar el sub-agente de catalogos
```

### Convencion de nombres

| Tipo de archivo | Patron | Contenido |
|---|---|---|
| Modelo | `model.txt` | Nombre del modelo de IA |
| Agente principal | `system-prompt.txt` | System prompt del agente coordinador |
| Sub-agente (prompt) | `subagent-{nombre}-prompt.txt` | Instruccion completa del sub-agente |
| Sub-agente (descripcion) | `subagent-{nombre}-descripcion.txt` | Breve descripcion de cuando activarlo (se usa como descripcion de la tool en n8n) |

---

## Arquitectura de Agentes

### 1. Agente Principal - "Jandrea" (Coordinador)

**Archivo:** `system-prompt.txt`

Es la asistente comercial que recibe todos los mensajes del cliente por WhatsApp. Actua como coordinadora: no busca productos ni envia imagenes por si misma, sino que enruta las peticiones a los sub-agentes correspondientes.

**Identidad:** Se presenta como parte del equipo ("tenemos", "hacemos"). Si le preguntan si es IA, responde: "Soy Jandrea, la asistente del taller".

**Responsabilidades principales:**
- Saludar y dar la bienvenida a los clientes
- Mantener conversacion fluida para entender que necesita el cliente
- Enrutar al sub-agente correcto cuando el cliente pide productos, fotos o catalogos
- Gestionar reglas de negocio (plazos, pagos 70/30, transferencias)
- Escalar a asesor humano SOLO en casos criticos

**Reglas criticas del agente principal:**
- NUNCA inventa precios, productos ni stock
- PROHIBIDO enviar links/URLs
- Mensajes cortos (max 4 lineas), sin markdown
- Maximo 1 pregunta por mensaje
- Cuando activa un sub-agente, NO escribe texto adicional

**Flujo de enrutamiento (cascada obligatoria):**
1. **PASO 1:** Si el cliente pide ver productos/fotos/precios de algo especifico → activa sub-agente `mostrar_catalogos`
2. **PASO 2:** Si el sub-agente de catalogos devuelve que NO existe catalogo para ese tema → activa sub-agente `mostrar_productos`

---

### 2. Sub-agente: mostrar_catalogos

**Archivos:**
- Prompt: `subagent-mostrar_catalogos-prompt.txt`
- Descripcion: `subagent-mostrar_catalogos-descripcion.txt`

**Se activa cuando:** El cliente menciona "catalogo" o pide ver productos/modelos/fotos de algo especifico.

**Descripcion de la tool (lo que ve el agente principal):**
> Usa esta herramienta SIEMPRE que el cliente mencione la palabra "catalogo" o "catalogos", o pida una lista general de productos por archivo. NO la usas si el cliente pide fotos o modelos especificos de un solo producto.

**Logica interna (3 condicionales):**

| Condicion | Accion |
|---|---|
| **A:** Cliente pidio un catalogo especifico y EXISTE | Ejecuta tool `enviar_catalogo_al_cliente` con la URL como parametro |
| **B:** Cliente pidio un catalogo especifico pero NO EXISTE | Ofrece alternativas de la lista, excluyendo los ya enviados |
| **C:** Peticion general ("tienes catalogo?") | Lista los catalogos disponibles para que el cliente elija |

**Herramientas que usa:**
- `obtener_catalogos_disponibles` - Lista todos los catalogos del sistema (se ejecuta SIEMPRE primero)
- `enviar_catalogo_al_cliente` - Envia un catalogo especifico al cliente via WhatsApp

**Reglas:**
- Siempre ejecuta `obtener_catalogos_disponibles` antes de responder (PASO 0)
- Nunca escribe URLs en el texto (viajan internamente en las tools)
- La respuesta final es solo texto limpio para WhatsApp

---

### 3. Sub-agente: mostrar_productos

**Archivos:**
- Prompt: `subagent-mostrar_productos-prompt.txt`
- Descripcion: `subagent-mostrar_productos-descripcion.txt`

**Se activa cuando:** El cliente pide ver fotos, modelos, ejemplos de un producto especifico, o pregunta precios de algo particular. Es la segunda opcion despues de que el sub-agente de catalogos falla.

**Descripcion de la tool (lo que ve el agente principal):**
> Usa esta herramienta cuando el cliente pida ver fotos, modelos, ejemplos, opciones de un producto especifico. NO la usas si la palabra clave es "catalogo".

**No es un chatbot:** Es una maquina de busqueda y envio de imagenes. Trabaja en 2 fases:

#### Fase 1 - Primera consulta:
1. Identifica la categoria con `DOCS_LISTA_CATEGORIAS`
2. Busca productos con `FETCH_PRODUCTOS_POR_CATEGORIA` (devuelve hasta 20 productos)
3. Selecciona 2 productos visualmente diferentes y envia sus imagenes con `enviar_imagen_http`
4. Envía un listado de hasta 7 productos (nombre + precio) en texto plano

#### Fase 2 - Cliente se interesa en un modelo especifico:
1. Identifica el producto elegido de los 20 guardados
2. Busca variaciones/similares (hasta 3, sin repetir imagenes)
3. Envia las imagenes con `enviar_imagen_http`
4. Mensaje corto de acompañamiento

**Herramientas que usa:**
- `DOCS_LISTA_CATEGORIAS` - Obtiene la lista de categorias disponibles
- `FETCH_PRODUCTOS_POR_CATEGORIA` - Busca productos por categoria (hasta 20)
- `enviar_imagen_http` - Envia una imagen al cliente via HTTP

**Reglas criticas:**
- NUNCA usa markdown para imagenes (no `![]`, no `[texto](url)`)
- NUNCA escribe URLs en el texto
- NUNCA repite imagenes ya enviadas
- Imagenes y texto siempre separados (primero imagenes, despues texto)

---

## Flujo Conversacional Completo

```
Cliente envia mensaje por WhatsApp
         │
         ▼
  ┌─────────────────────┐
  │  Agente Principal   │
  │    (Jandrea)        │
  └─────────┬───────────┘
            │
            ├── Saludo simple → Responde bienvenida
            │
            ├── Pide producto/fotos/precios → Activa sub-agente mostrar_catalogos
            │       │
            │       ├── Catalogo existe → Lo envia
            │       └── Catalogo NO existe → Activa sub-agente mostrar_productos
            │                                       │
            │                                       ├── Fase 1: 2 imagenes + listado
            │                                       └── Fase 2: 3 variantes del elegido
            │
            ├── Confirma pedido/pide cuenta → "Ya le ayudamos con eso"
            │
            ├── Fuera de tema → Redirige con calidez
            │
            └── Error critico / pide hablar con humano → Escala a asesor
```

---

## Reglas de Negocio

| Regla | Detalle |
|---|---|
| **Plazos** | 1-11 unidades: [dias configurables] laborables. Docenas o mas: 3 dias laborables. Urgentes: recargo adicional |
| **Pago** | 70% anticipo, 30% contra entrega. Se envian fotos del producto listo antes de cobrar el 30% |
| **Transferencia** | Cuando el cliente confirma pedido o pide numero de cuenta, responder: "Con mucho gusto, ya le ayudamos con eso" y detener la respuesta |
| **Escalado a humano** | Solo si: error critico de sistema, o el cliente lo exige explicitamente |
| **Problemas de envio** | Si el cliente dice que no cargo una imagen, reenviar con la tool, NO escalar |

---

## Notas para Modificaciones

- Para cambiar el modelo de IA, editar `model.txt`
- Para cambiar el comportamiento del agente principal, editar `system-prompt.txt`
- Para cambiar un sub-agente, editar sus archivos `subagent-{nombre}-prompt.txt` y `subagent-{nombre}-descripcion.txt`
- Para agregar un nuevo sub-agente, crear los archivos `subagent-{nombre}-prompt.txt` y `subagent-{nombre}-descripcion.txt`, y actualizar el enrutamiento en `system-prompt.txt`
- Las descripciones (`*-descripcion.txt`) son lo que el agente principal ve como descripcion de la tool en n8n
