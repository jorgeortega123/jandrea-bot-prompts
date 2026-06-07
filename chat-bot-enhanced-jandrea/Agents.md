Area de tabajo: 

N8N. Con nodos de aggent Ai nodo, que tienen:
Espacio de memoria, capacidad de poner tools y modelo de IA

Es la contruccion de un chat inteligente de Whastapp
que empieza desde el Clasficator.md donde esta la prompt que indica como debe clasificar la intencion del usuario para posteriormente delegar a un agente, la delagacion del agente solo se basa meramente en la intencion. en redirects.md se encuentra a como se delega a cada agente en base a su intencion. 

La estructura de carpetas te dice los agentes, y dentro de cada uno encontraras prompt_user y prompt_system donde indicara las prompt respectivas, opcionalmente encontraras otra carpetas debajo del agente, eso te indica que son SUB-AGENTES y encontrara un description.md que te indicara la descripcion que tiene ese subagente, para que el agente principal sepa cuando usarla. 

En cada carpeta que corresponda a un agente o subagente, puede haber la carpeta "tools" este te indicara que herramientas tiene disponible ese agente, en descrition.md dentro de la carpeta encontraras la descripcion que usa esa tool para indicarle al agente superior cuando debe usarla