# Chat Bot Jandrea - Guia de Arquitectura

## Area de trabajo

**Plataforma:** N8N con nodos de AI Agent.
Cada agente dispone de: espacio de memoria, tools y modelo de IA.

---

## Proyecto

Construccion de un chat inteligente para WhatsApp.

## Flujo principal

```
Mensaje de WhatsApp
       |
       v
[Clasificator.md]  -->  Clasifica la intencion del usuario
       |
       v
[Redirects.md]     -->  Redirige al agente correspondiente segun la intencion
       |
       v
[Agente]           -->  Procesa la solicitud y responde al usuario
  |
  +--> Puede delegar en un SUB-AGENTE si la tarea lo requiere
         |
         +--> Puede usar TOOLS para ejecutar acciones especificas
```

1. **Clasificador** (`Clasificator.md`): Recibe el mensaje y determina la intencion del usuario.
2. **Router** (`Redirects.md`): En base a la intencion, delega al agente correspondiente.
3. **Agente**: Procesa la solicitud. Puede usar tools o delegar en sub-agentes.
4. **Respuesta**: El agente devuelve la respuesta al usuario por WhatsApp.

---

## Estructura de carpetas

```
chat-bot-enhanced-jandrea/
├── Clasificator.md                # Prompt del clasificador de intenciones
├── Redirects.md                   # Logica de redireccion por intencion
│
├── informative_agent/             # AGENTE: Informacion general
│   ├── prompt_system.md           # System prompt del agente
│   ├── prompt_user.md             # User prompt del agente
│   ├── tools/                     # Herramientas del agente
│   │   ├── Info_jandrea/
│   │   │   └── description.md
│   │   └── Q&A/
│   │       └── description.md
│   └── mostrar_catalogos_subagent/  # SUB-AGENTE
│       ├── description.md         # Cuando usar este sub-agente
│       ├── prompt_system.md
│       └── tools/
│           ├── enviar_catalogo_al_cliente/
│           │   └── description.md
│           └── obtener_catalogos_disponibles/
│               └── description.md
│
└── quotation_agent/               # AGENTE: Cotizaciones y pedidos
    ├── prompt_system.md
    ├── prompt_user.md
    ├── tools/
    │   └── asignar_etiqueta/
    │       └── description.md
    └── mostrar_products_subagent/  # SUB-AGENTE
        ├── description.md
        ├── prompt_system.md
        └── tools/
            ├── DOCS_LISTA_CATEGORIAS/
            │   └── description.md
            ├── enviar_imagen_http/
            │   └── description.md
            └── FETCH_PRODUCTOS_POR_CATEGORIA/
                └── description.md
```

---

## Convenciones

### Agentes
- Cada carpeta `*_agent/` representa un agente.
- Contiene `prompt_system.md` y `prompt_user.md` con las instrucciones del agente.

### Sub-agentes
- Carpetas `*_subagent/` dentro de un agente.
- Tienen un `description.md` que indica **cuando** el agente principal debe delegar en ellos.

### Tools
- Carpeta `tools/` dentro de un agente o sub-agente.
- Cada tool tiene su propia carpeta con un `description.md` que describe su funcion, para que el agente sepa cuando utilizarla.

### Agregar un nuevo agente
1. Crear carpeta `nuevo_agente/`
2. Agregar `prompt_system.md` y `prompt_user.md`
3. Opcionalmente agregar `tools/` o `*_subagent/`
4. Actualizar `Clasificator.md` con la nueva intencion
5. Actualizar `Redirects.md` con la redireccion al nuevo agente
