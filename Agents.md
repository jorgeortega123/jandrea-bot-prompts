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
       +-- [comercial_agent]  -->  Productos, catalogos, captura de datos
       |      |
       |      +--> Consulta catalogos primero (catalogos_agent)
       |      |      +--> Si existe catalogo --> Propone enviarlo al cliente
       |      |      +--> Si NO existe --> Delega a productos_agent
       |      |
       |      +--> Captura: producto + cantidad + detalles
       |      +--> Deriva a asesor humano
       |
       +-- [info_jandrea_agent]  -->  Info de la empresa, direccion, horarios, pagos
```

1. **Clasificador** (`Clasificator.md`): Recibe el mensaje y determina la intencion del usuario.
2. **Router** (`Redirects.md`): En base a la intencion, delega al agente correspondiente.
3. **Agente comercial** (`comercial_agent/`): Captura informacion del cliente y deriva a asesor humano.
4. **Agente info** (`info_jandrea_agent/`): Responde preguntas generales sobre Jandrea.
5. **Respuesta**: El agente devuelve la respuesta al usuario por WhatsApp.

---

## Estructura de carpetas

```
chat-bot-enhanced-jandrea/
├── Clasificator.md                     # Prompt del clasificador de intenciones
├── Redirects.md                        # Logica de redireccion por intencion
│
├── comercial_agent/                    # AGENTE: Captura de datos y derivacion a asesor
│   ├── prompt_system.md
│   ├── prompt_user.md
│   ├── tools/
│   │   └── asignar_etiqueta/
│   │       └── description.md
│   ├── mostrar_catalogos_subagent/     # SUB-AGENTE: Catalogos (se consulta SIEMPRE primero)
│   │   ├── description.md
│   │   ├── prompt_system.md
│   │   └── tools/
│   │       ├── enviar_catalogo_al_cliente/
│   │       │   └── description.md
│   │       └── obtener_catalogos_disponibles/
│   │           └── description.md
│   └── mostrar_products_subagent/      # SUB-AGENTE: Productos (solo si no hay catalogo)
│       ├── description.md
│       ├── prompt_system.md
│       └── tools/
│           ├── DOCS_LISTA_CATEGORIAS/
│           │   └── description.md
│           ├── enviar_imagen_http/
│           │   └── description.md
│           └── FETCH_PRODUCTOS_POR_CATEGORIA/
│               └── description.md
│
└── info_jandrea_agent/                 # AGENTE: Info general de la empresa
    ├── prompt_system.md
    ├── prompt_user.md
    └── tools/
        ├── Info_jandrea/
        │   └── description.md
        └── Q&A/
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
