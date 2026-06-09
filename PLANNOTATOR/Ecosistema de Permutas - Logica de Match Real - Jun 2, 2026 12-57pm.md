---
created: 2026-06-02T15:57:26.691Z
source: plannotator
tags: [plannotator, manijacasas, ecosistema, permutas, logica, javascript, sql]
---

[[Plannotator Plans]]

# Ecosistema de Permutas - Logica de Match Real

## Contexto de Negocio

ManijaCasas es una **red social de propiedades** (tipo Instagram inmobiliario). No es un clasificado
tradicional. Su diferencial es:

- **Feed social**: las publicaciones se ordenan por actividad del propietario (ranking dinamico),
  no por fecha. Likes, comentarios, compartidos y actualizaciones del aviso determinan visibilidad.
- **Sin intermediarios**: propietario y comprador se conectan directamente.
- **Permutas como diferenciador estrategico**: la permuta no es un feature secundario.
  Es una de las 3 operaciones base (venta, alquiler, permuta) y tiene una fase de desarrollo
  completa dedicada (Fase 4, 60hs). El documento funcional dice: *"Match de permutas y
  notificaciones automaticas si hay coincidencias"*.

### El problema real

Hoy el flujo de permuta es 100% manual y ciego:
1. Un propietario publica su casa y marca "acepta permuta" + indica que acepta (ej: vehiculos)
2. Un comprador scrollea el feed, ve la propiedad, nota que acepta permuta
3. El comprador hace click en el boton "Permutar" (icono de flechas) en la card
4. Se abre el `SwapProposalDialog` donde arma una propuesta con lineas de permuta
5. La propuesta llega al propietario como notificacion

**Lo que falta**: No hay forma de saber **a quien vale la pena mandarle una propuesta**.
El comprador no sabe cuales propiedades aceptarian lo que el tiene. Y el propietario
no sabe que hay alguien con exactamente lo que busca.

La pagina `/permutas` ("Ecosistema de Permutas") fue disenada para resolver esto,
pero tiene datos mock y cero logica. El campo `matchFound` en el feed mapper esta
hardcodeado a `false`.

---

## Que debe resolver el match

En el contexto de red social, el match de permutas debe responder UNA pregunta:

> **"A quien le mando mi propuesta de permuta para que tenga chances reales de aceptarla?"**

Esto implica cruzar dos dimensiones:

| Dimension | Dato existente | Donde vive |
|-----------|----------------|------------|
| Lo que el propietario A **tiene** | `property.type` (house, apartment, etc.) | `properties.type` |
| Lo que el propietario A **acepta** | `property.swapLines` ([{category, type}]) | `properties.swap_lines` (JSONB) |
| Lo que el propietario B **tiene** | `property.type` | `properties.type` |
| Lo que el propietario B **acepta** | `property.swapLines` ([{category, type}]) | `properties.swap_lines` (JSONB) |

El match perfecto: A acepta lo que B tiene, Y B acepta lo que A tiene.
La sugerencia parcial: A acepta lo que B tiene (pero no sabemos si B acepta lo de A).

---

## Plan de Implementacion — Sugerencias Inteligentes en el Ecosistema

### Paso 1: Migracion — Indice GIN en swap_lines

**Archivo**: `src/database/migrations/0xx-add-gin-index-swap-lines.js`
**Que hace**: Crear indice GIN en la columna JSONB `swap_lines` para que las
queries de matching con operador `@>` sean performantes.

```javascript
module.exports = {
  async up(queryInterface) {
    await queryInterface.sequelize.query(`
      CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_properties_swap_lines_gin
      ON properties USING GIN (swap_lines jsonb_path_ops)
      WHERE swap_enabled = true;
    `);
  },
  async down(queryInterface) {
    await queryInterface.sequelize.query(`
      DROP INDEX CONCURRENTLY IF EXISTS idx_properties_swap_lines_gin;
    `);
  },
};
```

**Nota sobre transacciones**: Este proyecto usa Umzug (no Sequelize CLI) para
migraciones. Umzug **no** wrappea las migraciones en transacciones por defecto,
asique `CREATE INDEX CONCURRENTLY` funciona sin cambios especiales. No hace
falta agregar `transaction: false` ni configurar `migrationOptions`.

El indice parcial (`WHERE swap_enabled = true`) reduce el tamano del indice
porque solo indexa propiedades que efectivamente participan en permutas.

**Esfuerzo**: 0.5 dia

---

### Paso 2: Migracion — Tabla dismissed_swap_suggestions

**Archivo**: `src/database/migrations/0xx-create-dismissed-swap-suggestions.js`
**Que hace**: Guardar cuando un usuario descarta una sugerencia para no volver a mostrarla.

```sql
CREATE TABLE dismissed_swap_suggestions (
  id            SERIAL PRIMARY KEY,
  user_id       INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  property_id   INTEGER NOT NULL REFERENCES properties(id) ON DELETE CASCADE,
  suggested_property_id INTEGER NOT NULL REFERENCES properties(id) ON DELETE CASCADE,
  created_at    TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, property_id, suggested_property_id)
);
CREATE INDEX idx_dismissed_swap_user_prop ON dismissed_swap_suggestions(user_id, property_id);
```

**Modelo Sequelize**: `src/database/models/dismissed-swap-suggestion.model.ts`

**Registro del modelo**: Agregar `DismissedSwapSuggestion` en
`src/modules/properties/properties.module.ts` dentro de
`SequelizeModule.forFeature([..., DismissedSwapSuggestion])`.

**Esfuerzo**: 0.5 dia

---

### Paso 3: Verificacion previa — Taxonomia de tipos

**Antes de codear el algoritmo**, verificar que los valores de `property.type`
coinciden exactamente con los valores usados en `swapLine.type`.

**Archivos a cruzar**:
- `components/property-form/constants.ts` → `PERMUTA_TYPES` (valores que el usuario elige
  al configurar sus swapLines)
- `types/property-form.ts:527` → `mapPermutaLinesToSwapLines()` (traduce categorias
  pero pasa `type` directo)
- Valores reales de `properties.type` en la base de datos

**Posibles problemas**:
- El campo `type` de la propiedad podria guardar `'house'` pero `PERMUTA_TYPES`
  podria listar `'casa'` → el match fallaria silenciosamente
- Si hay discrepancia, agregar un mapping de normalizacion en el servicio
  de matching, o alinear los valores en `PERMUTA_TYPES`

**Accion**: Correr una query en dev `SELECT DISTINCT type FROM properties` y comparar
con los valores de `PERMUTA_TYPES`. Documentar el resultado antes de seguir.

**Esfuerzo**: 0.25 dia

---

### Paso 4: Verificacion previa — Campo status del modelo Property

**Resultado de la verificacion**: El modelo `Property` usa `paranoid: true`
(linea 31 del modelo). Esto significa que tiene un campo `deletedAt` y Sequelize
ORM excluye automaticamente los registros eliminados en queries ORM.

**Implicacion para raw SQL**: En las queries de matching, hay que agregar
`AND p.deleted_at IS NULL` manualmente porque `sequelize.query()` no aplica
los scopes de paranoid. Esto ya esta contemplado en el Paso 5.

No existe un campo `status = 'active'`. El filtro por estado se hace
exclusivamente via `deleted_at IS NULL`.

**Esfuerzo**: incluido en Paso 5
---

### Paso 5: Backend — Algoritmo de matching en PropertiesService

**Archivo**: `src/modules/properties/properties.service.ts`
**Nuevo metodo**: `findSwapSuggestions(propertyId: number, userId: number)`

**Logica del algoritmo**:

```
1. Obtener la propiedad del usuario (validar que es suya y tiene swapEnabled)
   → miPropiedad = { id, type, swapLines, price, city, province }
   Si la propiedad no es del usuario → 403 ForbiddenException
   Si swapEnabled = false → 400 BadRequestException

2. Query de MATCHES PERFECTOS (bidireccional, SIN paginacion SQL):
   SELECT p.*, u.first_name, u.avatar FROM properties p
   JOIN users u ON u.id = p.user_id
   WHERE p.swap_enabled = true
     AND p.user_id != :userId
     AND p.deleted_at IS NULL                         -- soft-delete safety
     -- Condicion 1: la otra propiedad acepta mi tipo
     AND p.swap_lines @> :miTipoComoSwapLine
     -- Condicion 2: yo acepto el tipo de la otra propiedad
     AND EXISTS (
       SELECT 1 FROM jsonb_array_elements(:misSwapLines) AS ml
       WHERE (ml->>'category') = 'property'
         AND (ml->>'type') = p.type
     )
   ORDER BY p.created_at DESC
   LIMIT 100

   Donde :miTipoComoSwapLine = '[{"category":"property","type":"<miPropiedad.type>"}]'

3. Guardar TODOS los IDs de matches perfectos en variable (sin paginar).

4. Query de SUGERENCIAS (unidireccional, SIN paginacion SQL):
   SELECT p.*, u.first_name, u.avatar FROM properties p
   JOIN users u ON u.id = p.user_id
   WHERE p.swap_enabled = true
     AND p.user_id != :userId
     AND p.deleted_at IS NULL
     AND p.swap_lines @> :miTipoComoSwapLine
     -- Excluir TODOS los matches perfectos (no solo una pagina)
     AND (CASE WHEN :hayMatchesPerfectos THEN p.id != ALL(:todosLosIdsPerfectos) ELSE TRUE END)
   ORDER BY p.created_at DESC
   LIMIT 100

   NOTA: Se usa `!= ALL(array)` en lugar de `NOT IN ()` para evitar el
   error de sintaxis de PostgreSQL cuando el array esta vacio.
   Alternativamente, usar `CASE WHEN` para omitir la condicion si no hay IDs.

5. Filtrar dismissed EN MEMORIA:
   - Cargar los IDs descartados: SELECT suggested_property_id
     FROM dismissed_swap_suggestions
     WHERE user_id = :userId AND property_id = :propertyId
   - Filtrar AMBAS listas removiendo los IDs descartados
   - Esto permite cachear las queries pesadas (2 y 4) sin
     que los dismiss invaliden el cache

6. Calcular SCORE para cada resultado (en memoria, sobre listas filtradas):
   - Base bidireccional: 80 puntos
   - Base unidireccional: 40 puntos
   - Bonus zona geografica: +10 (misma provincia/ciudad)
   - Bonus rango de precio: +10 (precio dentro de ±30% del mio)
   - Score final = min(100, base + bonuses)

7. Ordenar cada lista por score DESC.

8. NO paginar en el servidor. Devolver ambas listas completas (max 100 cada
   una post-filtrado). Razon: en un marketplace de nicho como ManijaCasas,
   la cantidad de propiedades con swap activo sera baja (decenas, no miles).
   El LIMIT 100 en SQL es un safety cap, no paginacion.
   Si en el futuro el volumen crece, se agrega paginacion por lista
   como mejora incremental.
```

**Por que NO paginar en el servidor (MVP)**:
- El numero de propiedades con `swapEnabled = true` en un marketplace de nicho
  argentino sera bajo al inicio (decenas, quizas cien).
- Paginar en SQL y luego filtrar dismissed en memoria produce paginas
  de tamano inconsistente (se pierde la garantia de "20 resultados por pagina").
- Paginar ANTES de excluir matches perfectos de las sugerencias haria
  que algunos matches perfectos aparezcan como sugerencias en paginas
  posteriores.
- Solucion simple: traer todo (cap 100), filtrar en memoria, devolver completo.
  El frontend puede implementar infinite scroll local si la lista es larga.

**Nota sobre vehiculos/embarcaciones**:
Las swapLines pueden tener categorias `property`, `vehicle`, `boat`.
El match bidireccional solo aplica para category `property` (porque vehiculos
y embarcaciones no se publican como propiedades en la plataforma).
Para `vehicle`/`boat` solo hay match unidireccional ("esta persona acepta
vehiculos, quizas te interese proponerle").

**Nota sobre raw SQL con Sequelize**:
Este proyecto no tiene precedentes de `sequelize.query()` con operadores JSONB.
Se debe usar `sequelize.query()` con `replacements` para parametros escalares,
y construir los JSONB como strings literales para los operadores `@>` y
`jsonb_array_elements`. Ejemplo:
```typescript
const miTipoComoSwapLine = JSON.stringify([
  { category: 'property', type: propiedad.type }
]);
const misSwapLines = JSON.stringify(propiedad.swapLines);

await sequelize.query<PropertyRow>(
  `SELECT p.*, u.first_name, u.avatar
   FROM properties p JOIN users u ON u.id = p.user_id
   WHERE p.swap_enabled = true
     AND p.user_id != :userId
     AND p.deleted_at IS NULL
     AND p.swap_lines @> :miTipo::jsonb
     ...`,
  {
    replacements: { userId, miTipo: miTipoComoSwapLine },
    type: QueryTypes.SELECT,
  }
);
```
Para el `!= ALL(array)`, construir la clausula dinamicamente:
- Si hay IDs de matches perfectos: agregar `AND p.id != ALL(ARRAY[${ids.join(',')}])`
- Si no hay IDs: omitir la clausula (no agregar condicion)

**Nota sobre `NOT IN` con array vacio**:
PostgreSQL falla con `NOT IN ()` cuando la lista esta vacia.
Construir dinamicamente: si hay IDs, agregar la exclusion; si no, omitirla.

**Esfuerzo**: 2 dias

---

### Paso 6: Backend — DTO de respuesta

**Archivo**: `src/modules/properties/dto/swap-suggestion.dto.ts`

```typescript
export class SwapSuggestionDto {
  property: {
    id: number;
    title: string;
    type: string;
    price: number;
    priceCurrency: string;
    location: { city: string; province: string };
    images: { url: string }[];
    swapLines: { category: string; type: string }[];
    owner: { id: number; firstName: string; avatar?: string };
  };
  matchScore: number;          // 0-100
  matchType: 'perfect' | 'suggestion';
  matchingLines: {             // cuales lineas causaron el match
    theirLine: { category: string; type: string };  // lo que ellos aceptan (= lo mio)
    myLine?: { category: string; type: string };     // lo mio que matchea con ellos (solo en perfect)
  }[];
}

export class SwapSuggestionsResponseDto {
  perfectMatches: SwapSuggestionDto[];  // max ~100, ordenados por score DESC
  suggestions: SwapSuggestionDto[];     // max ~100, ordenados por score DESC
  myProperty: {
    id: number;
    title: string;
    type: string;
    swapLines: { category: string; type: string }[];
  };
  // Sin paginacion en MVP. Ambas listas se devuelven completas.
  // El frontend puede paginar localmente si es necesario.
}

**Esfuerzo**: 0.5 dia

---

### Paso 7: Backend — Endpoint y controller

**Archivo**: `src/modules/properties/properties.controller.ts`

Nuevos endpoints (agregar ANTES de las rutas parametrizadas `:id`,
al lado de los otros endpoints `swaps/*` existentes en lineas 322-418):

```typescript
// GET /api/properties/swaps/suggestions/:propertyId
@Get('swaps/suggestions/:propertyId')
@UseGuards(JwtAuthGuard)
@ApiOperation({ summary: 'Obtener sugerencias de permuta para una propiedad' })
@ApiResponse({ status: 200, type: SwapSuggestionsResponseDto })
async getSwapSuggestions(
  @Param('propertyId', ParseIntPipe) propertyId: number,
  @Req() req,
): Promise<SwapSuggestionsResponseDto> {
  return this.propertiesService.findSwapSuggestions(propertyId, req.user.id);
}

// POST /api/properties/swaps/suggestions/:propertyId/dismiss/:suggestedId
@Post('swaps/suggestions/:propertyId/dismiss/:suggestedId')
@UseGuards(JwtAuthGuard)
@ApiOperation({ summary: 'Descartar una sugerencia de permuta' })
@HttpCode(HttpStatus.NO_CONTENT)
async dismissSwapSuggestion(
  @Param('propertyId', ParseIntPipe) propertyId: number,
  @Param('suggestedId', ParseIntPipe) suggestedId: number,
  @Req() req,
): Promise<void> {
  return this.propertiesService.dismissSwapSuggestion(
    propertyId, suggestedId, req.user.id,
### Paso 8: Backend — Cache en Redis

**Archivo**: `src/modules/properties/properties.service.ts` (dentro de `findSwapSuggestions`)

**Estrategia de cache**:
- Cachear los **resultados crudos** de las queries SQL (IDs + datos de propiedades)
  SIN filtrar dismissed. Asi el cache se mantiene valido independientemente
  de los dismiss.
- Al servir la respuesta, cargar la lista de dismissed desde la tabla y
  filtrar en memoria. La tabla `dismissed_swap_suggestions` es pequena
  y tiene indice, asi que es rapida.

**Configuracion**:
- Key: `swap:suggestions:{propertyId}:raw`
- TTL: 24 horas
- Invalidacion: **No existe un evento `property.updated` en el sistema actual**.
  Los eventos existentes son `property.liked`, `property.viewed`, y eventos de swap.
  Por lo tanto, la invalidacion del cache se hara de dos formas:
  1. **Al actualizar una propiedad del usuario**: agregar un hook directamente
     en el metodo `update()` de `PropertiesService` que elimine la clave
     `swap:suggestions:{propertyId}:raw` de Redis cuando `swapLines` cambie.
     Esto se implementa como una simple llamada a `redis.del()` dentro del servicio.
  2. **Para propiedades de terceros**: no invalidar; el TTL de 24hs cubre
     la eventualidad de nuevas propiedades.

**Justificacion del TTL**:
El inventario de propiedades no cambia minuto a minuto. Un delay de hasta
24 horas para ver propiedades nuevas de terceros es aceptable para un MVP.
Si el usuario quiere forzar un refresh, puede recargar la pagina
(el frontend puede pasar un query param `?fresh=true` que saltee el cache).

**Nota**: Redis ya esta en el stack (se usa para sesiones/notificaciones).
No requiere infra nueva.

**Esfuerzo**: 0.5 dia
---

### Paso 9: Backend — Tests unitarios

**Archivo**: `src/modules/properties/properties.service.spec.ts`

Casos a cubrir:
- Match perfecto: A tiene casa, acepta departamentos. B tiene departamento,
  acepta casas → match bidireccional, score >= 80
- Sugerencia: A tiene departamento. B acepta departamentos pero A no acepta lo de B
  → solo sugerencia unidireccional, score < 80
- Sin matches: A tiene casa, nadie acepta casas → listas vacias
- Propiedad ajena: intentar consultar sugerencias de propiedad de otro → 403
- Propiedad sin swap: consultar propiedad con swapEnabled=false → 400
- Descarte: despues de dismiss, la propiedad no aparece en sugerencias
- Bonus geografico: misma provincia → score mayor
- Bonus precio: precio similar → score mayor
- Paginacion: NO se pagina en servidor; verificar que el endpoint devuelve
  ambas listas completas y el frontend las maneja
- Propiedades eliminadas (soft-delete): no aparecen en resultados
- Solo categorias `property` generan match bidireccional;
  `vehicle`/`boat` solo aparecen en sugerencias
- Array vacio de matches perfectos: la query de sugerencias no falla
  con NOT IN () vacio

---

### Paso 10: Frontend — Hook useSwapSuggestions

**Archivo**: `hooks/use-swap-suggestions.ts`

```typescript
export function useSwapSuggestions(propertyId: number | null) {
  // SWR fetch de GET /api/properties/swaps/suggestions/:propertyId
  // Solo se ejecuta si propertyId es truthy y session es authenticated
  // Retorna: { data: SwapSuggestionsResponseDto, isLoading, error, mutate }
}

export function useDismissSuggestion() {
  // Mutation que llama POST .../dismiss/:suggestedId
  // Optimistic update: remueve la sugerencia del cache SWR local
  //   antes de que el server responda
  // On error: rollback (volver a agregar la sugerencia al cache)
}
```

**Esfuerzo**: 0.5 dia

---

### Paso 11: Frontend — Componente SwapSuggestionCard

**Archivo**: `components/swap-suggestion-card.tsx`

Componente nuevo dedicado (no reutilizar `PropertyCard` completo porque
las sugerencias tienen info distinta: score, tipo de match, lineas que matchean).

Props:
```typescript
interface SwapSuggestionCardProps {
  suggestion: SwapSuggestionDto;
  onPropose: (propertyId: number) => void;   // abre SwapProposalDialog
  onDismiss: (propertyId: number) => void;   // descarta sugerencia
}
```

El componente muestra:
- Imagen con aspect ratio 16/10 (mismo que el mock actual)
- Score en circulo (verde >80, amarillo >50, gris <50)
- Tipo de match: badge "Match perfecto" (verde) o "Sugerencia" (amarillo)
- Lineas de match como pills (reutilizando estilos de `PropertySwap`)
- Info del propietario (avatar + nombre)
- Precio de la propiedad sugerida
- Diferencia de precio respecto a la mia (ej: "+120k USD" como en el mock)
- Boton "Proponer permuta" (primario)
- Boton "Descartar" (icono X, ghost)

**Esfuerzo**: 1 dia

---

### Paso 12: Frontend — Refactor de app/permutas/page.tsx

**Archivo**: `app/permutas/page.tsx`
**Cambio**: Reemplazar completamente el contenido mock por datos reales.

**Estructura nueva de la pagina**:

```
┌──────────────────────────────────────────────────────┐
│ [Sticky Header] Ecosistema de Permutas               │
│ Gestiona tus intercambios y matches de propiedades   │
├──────────────────────────────────────────────────────┤
│                                                      │
│ Tu propiedad                                         │
│ [Select: mis propiedades con swap activo]             │
│                                                      │
│ ─────────────────────────────────────────────         │
│                                                      │
│ Matches perfectos (X)                    🟢           │
│ Estas propiedades aceptan lo tuyo Y vos              │
│ aceptas lo de ellos                                  │
│                                                      │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐                 │
│ │ Card 1  │ │ Card 2  │ │ Card 3  │                 │
│ │ 95%     │ │ 90%     │ │ 85%     │                 │
│ └─────────┘ └─────────┘ └─────────┘                 │
│                                                      │
│ ─────────────────────────────────────────────         │
│                                                      │
│ Sugerencias (X)                          🟡           │
│ Estas propiedades aceptan lo que tenes,              │
│ podrias proponerles una permuta                      │
│                                                      │
│ ┌─────────┐ ┌─────────┐                              │
│ │ Card 4  │ │ Card 5  │                              │
│ │ 60%     │ │ 45%     │                              │
│ └─────────┘ └─────────┘                              │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Selector de propiedad**:
**Selector de propiedad**:
- Usa `useMyProperties()` (ya existe)
- Filtra solo propiedades con `swapEnabled = true`
- Al cambiar seleccion, dispara `useSwapSuggestions(selectedPropertyId)`

**Estados de la pagina**:
- **No autenticado**: redirect a `/login` (mismo patron que `app/swaps/page.tsx`)
- **Sin propiedades con swap**: "No tenes propiedades con permuta activa.
  Edita una de tus propiedades y activa la opcion 'Acepta permuta'." + CTA a dashboard
- **Propiedad seleccionada, sin resultados (matches perfectos vacio)**:
  Si el usuario solo acepta vehiculos/embarcaciones, mostrar texto explicativo:
  *"Los matches perfectos requieren que ambas partes publiquen propiedades
  compatibles. Tus preferencias incluyen vehiculos/embarcaciones que no se
  publican como propiedades en la plataforma, pero podes ver sugerencias abajo."*
- **Propiedad seleccionada, sin resultados (ambas listas vacias)**:
  "No encontramos matches todavia. Cuando alguien publique una propiedad
  compatible, aparecera aca."
- **Loading**: skeleton de cards (reutilizar `PropertyCardSkeleton`)
- **Error**: mensaje de error con boton "Reintentar"

**Nota**: El campo `price` en el modelo `Property` tiene `allowNull: false`,
asique siempre hay un precio disponible para el calculo del bonus de rango.
No hace falta manejar el caso de precio null.

**Esfuerzo**: 2-3 dias
---

### Paso 13: Frontend — Conectar SwapProposalDialog

**Archivo**: `app/permutas/page.tsx` (integracion) + cambio menor en
`components/swap-proposal-dialog.tsx`

El `SwapProposalDialog` ya existe y funciona. Ajustes:

1. **En la pagina de permutas**:
   - Importar `SwapProposalDialog`
   - Mantener estado `selectedSuggestionPropertyId` y `isSwapDialogOpen`
   - Cuando el usuario hace click en "Proponer permuta" en una card,
     abrir el dialog con `propertyId = suggestion.property.id`

2. **Cambio menor en `SwapProposalDialog`**:
   - Agregar prop opcional `suggestedFromPropertyId?: number`
   - Si se pasa, pre-seleccionar esa propiedad del usuario como la primera
     linea de la propuesta (el dialog ya carga `useMyProperties()` internamente)
   - Esto mejora la UX porque el usuario no tiene que buscar manualmente
     su propiedad en el dropdown; ya sabe cual quiere ofrecer porque
     la eligio en el selector de la pagina de permutas

**Esfuerzo**: 0.5 dia

---

### Paso 14: Frontend — Swagger y documentacion

- Los decoradores `@ApiOperation`, `@ApiResponse` y los DTOs ya cubren
  la documentacion automatica de Swagger
- Agregar tag `permutas` al controller o reutilizar el tag `properties`
- Descripciones en espanol segun la convencion del proyecto (AGENTS.md)

**Esfuerzo**: 0.25 dia

---

## Resumen de esfuerzo

| Paso | Tarea | Esfuerzo |
|------|-------|----------|
| 1 | Migracion indice GIN (sin transaccion) | 0.5d |
| 2 | Migracion tabla dismissed + modelo + registro en modulo | 0.5d |
| 3 | Verificacion taxonomia de tipos | 0.25d |
| 4 | Verificacion campo status/paranoid | (incluido en 5) |
| 5 | Algoritmo de matching (sin paginacion SQL, filtrado en memoria) | 2d |
| 6 | DTO de respuesta (sin meta de paginacion) | 0.5d |
| 7 | Endpoint y controller (sin page/limit) | 0.5d |
| 8 | Cache Redis (resultados crudos, filter dismissed en app) | 0.5d |
| 9 | Tests unitarios (12 casos, incluye array vacio) | 1d |
| 10 | Hook useSwapSuggestions + useDismissSuggestion | 0.5d |
| 11 | SwapSuggestionCard | 1d |
| 12 | Refactor pagina permutas (estados, UX vehiculos/embarcaciones) | 2.5d |
| 13 | Conectar SwapProposalDialog + prop suggestedFromPropertyId | 0.5d |
| 14 | Swagger/docs | 0.25d |
| **Total** | | **~10.5 dias** |

---

## Orden de ejecucion

**Bloque 0 — Verificaciones (dia 1 manana)**:
Pasos 3 → 4
(Confirmar taxonomia de tipos y campo status antes de codear)

**Bloque 1 — Backend (dias 1-5)**:
Pasos 1 → 2 → 6 → 5 → 7 → 8 → 9
(Migraciones primero, luego DTOs, luego algoritmo con las verificaciones
resueltas, endpoint, cache, tests)

**Bloque 2 — Frontend (dias 6-10.5)**:
Pasos 10 → 11 → 12 → 13 → 14
(Hook primero, luego el componente card, luego la pagina que los une,
luego la integracion con el dialog, y finalmente docs)

Los bloques pueden paralelizarse parcialmente si hay dos personas.

---

## Inconsistencias resueltas (todas)

| # | Problema | Resolucion |
|---|----------|------------|
| 1 | `CREATE INDEX CONCURRENTLY` necesita transaccion especial | No aplica: Umzug no wrappea en transacciones. Funciona directo |
| 2 | Cache Redis contradice filtro de dismissed en SQL | Paso 5/8: cachear resultados crudos, filtrar dismissed en memoria |
| 3 | Invalidar cache con `property.updated` evento que no existe | Paso 8: invalidar con hook directo en `update()` del service, no con evento |
| 4 | `p.status = 'active'` no existe; `paranoid: true` confirmado | Paso 4/5: usar `deleted_at IS NULL` en raw queries |
| 5 | Taxonomia `property.type` vs `swapLine.type` | Paso 3: verificacion explicita antes de codear |
| 6 | Paginacion SQL + filtrado dismissed = paginas inconsistentes | Paso 5: sin paginacion SQL; traer todo (cap 100), filtrar en memoria |
| 7 | Matches perfectos vacio si solo acepta vehiculos | Paso 12: texto explicativo especifico para ese caso |
| 8 | Modelo dismissed no registrado en modulo | Paso 2: registro explicito en `properties.module.ts` |
| 9 | SwapProposalDialog no pre-selecciona propiedad origen | Paso 13: prop `suggestedFromPropertyId` nueva |
| 10 | `NOT IN ()` con array vacio = error SQL | Paso 5: construir clausula dinamicamente, omitir si no hay IDs |
| 11 | `NOT IN` con IDs paginados filtra incompleto | Paso 5: sin paginacion SQL; se usan TODOS los IDs de matches perfectos |
| 12 | No hay patron de `sequelize.query()` con JSONB | Paso 5: documentar uso de string literal con `replacements` |
| 13 | Auth redirect es `/login` no `/auth/login` | Paso 12: corregido a `/login` |
| 14 | Migracion Umzug no necesita `transaction: false` | Paso 1: nota actualizada |