Obtiene los productos de una categoría específica usando el endpoint GET /products/by-category-code/:categoryId. RECUERDA DEBES MANDA EL CATEGORYID Que son aqueelos tipo: "1a', "5a", etc. No mandes el nombre de la categoria

CUÁNDO USAR: Úsala cuando el usuario quiera ver, listar o consultar los productos de una categoría concreta.

PRERREQUISITOS OBLIGATORIOS antes de llamar a este endpoint:
1. Primero debes haber consultado el DOCS de CATEGORÍAS para conocer las categorías disponibles y sus IDs. (HHTP REQUEST; LISTA DE CATEGORIAS)
2. Debes tener un categoryId válido (obtenido del DOCS de categorías). Sin categoryId no se puede llamar a este endpoint.

ESTRUCTURA DEL PRODUCTO que devuelve este endpoint (array "productos"):
{
  id: number,
  nombre: string,           // nombre del producto
  descripcion: string,      // descripción del producto
  identificador: string,    // slug/URL del producto (úsalo para el link, NO el id)
  precio: number,           // precio base
  descuento: number,        // porcentaje de descuento
  categoryId: number,
  variants: [               // variantes del producto (tamaños, colores, etc.)
    {
      id: number,
      precio: number,
      stock: number,
      images: [             // imágenes de esta variante
        {
          src: string,      // URL de la imagen (usa SIEMPRE .src, NO .url)
          isVideo: boolean, // si es true, es un video — NO incluir como imagen
          needContrast: boolean
        }
      ]
    }
  ]
}

RESPUESTA del endpoint: { productos: [...], hasNextPage: boolean }

PARÁMETROS:
- categoryId (OBLIGATORIO, en la URL): ID de la categoría cuyos productos se quieren obtener.
- page (opcional, query param): Número de página. Default: 1.
- limit (opcional, query param): Cantidad de productos por página. Default: 5.
- sortByPrice (opcional, query param): Ordenar por precio. Valores: "asc" o "desc".
- sortByDate (opcional, query param): Ordenar por fecha de creación. Valores: "asc" o "desc".
- all (opcional, query param): Si se envía cualquier valor, devuelve todos los productos sin paginar.

IMPORTANTE: Cuando el usuario quiera ver productos con imágenes, emite al FINAL de tu respuesta el bloque [PRODUCTOS_CARDS]{JSON del array productos}[PRODUCTOS_CARDS]{
  "productos":[
    {
      "title":"Caja decorativa",
      "description":"...",
      "identificador":"abc",
      "variants":[
        {
          "price":4.25,
          "images":[{"src":"url"}]
        }
      ]
    }
  ]
}[/PRODUCTOS_CARDS]