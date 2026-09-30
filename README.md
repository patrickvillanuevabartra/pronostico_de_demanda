# Simulador de pronósticos de demanda

Página web (un solo archivo, sin instalación) para que cada grupo cargue la demanda proyectada de su proyecto,
compare los modelos de la plantilla `Modelos_Pronosticos_ST.xlsx` más la tasa media de crecimiento, y obtenga
el modelo recomendado con su justificación. Opcionalmente envía los resultados a **QuintaDB**.

## Cómo se conecta

```
Navegador del grupo (GitHub Pages)  ──►  Google Apps Script (Code.gs)  ──►  QuintaDB
        index.html                        guarda la clave de la API         base de datos
```

La clave de la API de QuintaDB da acceso total (crear, editar y borrar). Por eso **no se pone en `index.html`**,
que es público: queda solo en las propiedades del script.

## Archivos

| Archivo | Uso |
|---|---|
| `index.html` | Toda la aplicación. Se publica en GitHub Pages. |
| `Code.gs` | Script de Apps Script que recibe los envíos y los guarda en QuintaDB. |

## 1. Publicar en GitHub Pages

1. Cree un repositorio nuevo en GitHub (por ejemplo `simulador-pronosticos`).
2. Suba `index.html` a la raíz.
3. En **Settings > Pages**, elija *Deploy from a branch*, rama `main`, carpeta `/ (root)`.
4. En uno o dos minutos la página queda en `https://SU-USUARIO.github.io/simulador-pronosticos/`.

Sin la sección 2, la página ya funciona: carga datos, simula y descarga el informe en Excel.

## 2. Guardar los resultados en QuintaDB (opcional)

1. En QuintaDB, copie su clave de API (menú **API**, arriba a la derecha).
2. Entre a [script.google.com](https://script.google.com), cree un **proyecto nuevo** y pegue el contenido de `Code.gs`.
3. En **Configuración del proyecto (engranaje) > Propiedades del script**, agregue:

   | Propiedad | Valor |
   |---|---|
   | `QDB_API_KEY` | su clave de la API de QuintaDB |
   | `TOKEN` | una clave a su elección (la usará también en `index.html`) |
   | `QDB_APP_ID` | *opcional*: ID de una base ya creada. Si no la pone, el script crea una |

4. En el editor, seleccione la función **`setup`** y pulse **Ejecutar**. Autorice los permisos.
   Se crea la base **«Pronosticos de demanda»** con tres tablas y todos sus campos:
   - **Resultados:** una fila por envío (grupo, proyecto, patrón, modelo elegido, errores, justificación, observaciones y la serie de demanda).
   - **Comparacion:** el ranking completo de modelos de cada envío.
   - **Pronostico:** valores futuros del modelo elegido con su banda.
5. **Implementar > Nueva implementación > Aplicación web**:
   - Ejecutar como: **Yo**
   - Quién tiene acceso: **Cualquier persona**
6. Copie la URL que termina en `/exec`. Al abrirla en el navegador debe mostrar `"ok":true` y `"configurado":true`.
7. En `index.html`, busque `const CONFIG = { ENDPOINT: '', TOKEN: '' };` y complete:

```js
const CONFIG = { ENDPOINT: 'https://script.google.com/macros/s/XXXXXXXX/exec', TOKEN: 'la misma clave de Script Properties' };
```

8. Suba de nuevo `index.html`. Aparece el botón **Enviar resultados al docente**.

Si algo falla en el paso 4, ejecute `probarConexion` para verificar que la clave funciona: debe listar sus bases en el registro de ejecución.

## Cómo se ven los envíos

- Cada envío tiene un `ID` que se repite en las tres tablas, para enlazar la fila de **Resultados** con sus filas de **Comparacion** y **Pronostico**.
- Reintentos por mala conexión no duplican registros (mismo `ID`).
- Si un grupo cambia sus observaciones y vuelve a enviar, se guarda como **un envío nuevo**. Para ver el último de cada grupo, ordene **Resultados** por `Fecha`.

## Notas

- El `TOKEN` queda visible en el código de la página. Sirve para frenar envíos accidentales o de spam, no como seguridad fuerte. La clave de QuintaDB, en cambio, nunca sale del script.
- Si modifica `Code.gs` después de implementarlo, cree una **nueva versión** (Implementar > Administrar implementaciones > Editar > Nueva versión).
- Los cálculos se hacen en el navegador de cada grupo. Solo se envía información si el grupo pulsa el botón de envío.
- La página usa Google Fonts y la librería SheetJS desde cdnjs, así que necesita conexión a internet.
