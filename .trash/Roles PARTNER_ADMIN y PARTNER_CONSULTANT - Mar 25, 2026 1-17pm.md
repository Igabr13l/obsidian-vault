---
created: 2026-03-25T16:17:13.196Z
source: plannotator
tags: [plannotator, hubeet, roles, partner_admin, partner_consultant, javascript, jsx]
---

[[Plannotator Plans]]


# Plan: Roles PARTNER_ADMIN y PARTNER_CONSULTANT

## Resumen de Decisiones
- **PARTNER** se renombra a **PARTNER_ADMIN** (migración de datos existentes)
- Se agrega **PARTNER_CONSULTANT** como nuevo permiso
- Se reutiliza `partnerUserMapper` para asignar organizaciones a consultores
- PARTNER_CONSULTANT solo ve `/partner/organization` (filtrado a sus orgs asignadas)
- PARTNER_ADMIN ve `/partner/organization`, `/partner/subscriptions`, `/partner/access`

---

## FASE 1: Base de Datos (hubeet-database-manager)

### 1.1 Migración: Renombrar PARTNER → PARTNER_ADMIN y agregar PARTNER_CONSULTANT
**Archivo nuevo:** `hubeet-database-manager/migrations/20260325120000-rename-partner-add-consultant-permission.js`

- ALTER TABLE `HUBEET_SUPPORT_userPermissions` → extender ENUM del campo `permission` para incluir `PARTNER_ADMIN` y `PARTNER_CONSULTANT`
- UPDATE todos los registros con `permission = 'PARTNER'` → `permission = 'PARTNER_ADMIN'`
- Eliminar el valor `PARTNER` del ENUM (o dejarlo deprecado si MySQL no permite eliminarlo fácilmente de ENUMs)
- Patrón a seguir: copiar estructura de `20260224120000-add-costs-manager-permission.js`

### 1.2 Migración: Renombrar PARTNER → PARTNER_ADMIN en ROLES de admin_users
**Mismo archivo de migración o separado**

- ALTER TABLE `HUBEET_ADMIN_users` → extender ENUM del campo `role` para incluir `PARTNER_ADMIN` y `PARTNER_CONSULTANT` (si aplica)
- Nota: El campo `role` en `HUBEET_ADMIN_users` usa `ROLES` (ADMIN, ANONYMOUS, DEFAULT, SUPERADMIN), NO `PERMISSIONS`. Evaluar si es necesario agregar los nuevos roles aquí o si se maneja exclusivamente vía la tabla de permisos.

---

## FASE 2: Backend (hubeet-backend-support)

### 2.1 Actualizar constantes de permisos
**Archivo:** `modules/permissions/constants.js`
```javascript
// Agregar:
PARTNER_ADMIN: "PARTNER_ADMIN",
PARTNER_CONSULTANT: "PARTNER_CONSULTANT",
// Eliminar o deprecar:
// PARTNER: "PARTNER", ← remover
```

### 2.2 Actualizar constantes de auth
**Archivo:** `modules/auth/constants.js`
```javascript
// En AUTH_PERMISSIONS:
// Reemplazar PARTNER por:
PARTNER_ADMIN: "PARTNER_ADMIN",
PARTNER_CONSULTANT: "PARTNER_CONSULTANT",
```

### 2.3 Actualizar GraphQL Controller de Partner Organizations
**Archivo:** `modules/partner/organizations/graphql/graphqlController.js`

- Cambiar todos los `[AUTH_PERMISSIONS.PARTNER]` por `[AUTH_PERMISSIONS.PARTNER_ADMIN, AUTH_PERMISSIONS.PARTNER_CONSULTANT]` en las queries de lectura (getPartnerOrganizationsCount, getPartnerOrganizationByUuid, getPartnerProxyUsers, etc.)
- Para mutations administrativas (createPartnerProxyUser, syncPartnerProxyUsers, togglePartnerPermission, resendWelcomeEmail, createOrganization): restringir a `[AUTH_PERMISSIONS.PARTNER_ADMIN]` solamente

### 2.4 Filtrar organizaciones para PARTNER_CONSULTANT
**Archivo:** `modules/partner/organizations/graphql/GQLResolvers/GQLQueries/getPartnerOrganizationsCount/resolver.js`

- Detectar si el usuario actual es PARTNER_CONSULTANT (chequeando sus permisos en contexto)
- Si es PARTNER_CONSULTANT: agregar filtro adicional al `businessFilter` que cruce con `HUBEET_PARTNER_USER_MAPPER` para solo devolver orgs donde el usuario tiene mapping
- Si es PARTNER_ADMIN: comportamiento actual (todas las orgs del partner)

**Archivo:** `modules/partner/organizations/db/dbService.js`
- Agregar lógica en `_processPartnerFilters` o crear método `_processConsultantFilters` que haga JOIN con `PartnerUserMapperModel` filtrando por `proxyUserId = currentUserId`

### 2.5 Actualizar resolver getPartnerOrganizationUsersWithPartnerRole
**Archivo:** `modules/partner/organizations/graphql/GQLResolvers/GQLQueries/getPartnerOrganizationUsersWithPartnerRole/resolver.js`

- Actualmente filtra usuarios con `permission === PERMISSIONS.PARTNER`
- Cambiar para aceptar un argumento opcional `role` (PARTNER_ADMIN o PARTNER_CONSULTANT)
- Filtrar usuarios según el permiso solicitado
- Esto alimenta el modal: si el caller es PARTNER_ADMIN → devolver solo PARTNER_ADMIN users. Si es PARTNER_CONSULTANT → devolver solo PARTNER_CONSULTANT users.

### 2.6 Actualizar togglePartnerPermission
**Archivo:** `modules/partner/organizations/graphql/GQLResolvers/GQLMutations/togglePartnerPermission/resolver.js`
**Archivo:** `modules/users/service.js` (método `togglePartnerPermission`)

- Aceptar un nuevo argumento `permissionType` ("PARTNER_ADMIN" | "PARTNER_CONSULTANT")
- Validar que solo PARTNER_ADMIN puede ejecutar esta mutación
- Actualizar lógica para agregar/quitar el permiso indicado

### 2.7 Actualizar Passport Partner Strategy
**Archivo:** `modules/auth/strategies/partnerStrategy/index.js`

- Cambiar la verificación de `PERMISSIONS.PARTNER` a `PERMISSIONS.PARTNER_ADMIN` o `PERMISSIONS.PARTNER_CONSULTANT`
- Ambos roles pueden hacer login como proxy user (el consultor solo en orgs asignadas, esto ya está validado por `canLoginAsProxyUser`)

### 2.8 Actualizar authenticateLogin (session response)
**Archivo:** `modules/auth/restController.js`

- En `authenticateLogin`, los permisos se mapean como `arrayPermissions = userSupport.permissions.map(p => p.permission)`
- Esto ya funcionará automáticamente con los nuevos nombres de permisos
- Solo verificar que no haya hardcodes de "PARTNER" en la construcción de la sesión

---

## FASE 3: Frontend (hubeet-frontend-platform)

### 3.1 Actualizar constantes de permisos
**Archivo:** `src/hubeet/contrants/permissions.jsx`
```javascript
// Reemplazar:
// PARTNER: { key: 'PARTNER', app: 'support' }
// Por:
PARTNER_ADMIN: { key: 'PARTNER_ADMIN', app: 'support' },
PARTNER_CONSULTANT: { key: 'PARTNER_CONSULTANT', app: 'support' },
```

**Archivo:** `src/modules/admin/components/PrivateRoute/Constants/index.jsx`
```javascript
// Reemplazar PARTNER por:
PARTNER_ADMIN: 'PARTNER_ADMIN',
PARTNER_CONSULTANT: 'PARTNER_CONSULTANT',
```

**Archivo:** `src/modules/auth/constants/index.jsx`
```javascript
export const ROLES = {
  PARTNER_ADMIN: 'PARTNER_ADMIN',
  PARTNER_CONSULTANT: 'PARTNER_CONSULTANT'
};
```

### 3.2 Actualizar getPrincipalRole
**Archivo:** `src/modules/auth/services/useLoginService.jsx`

- Reemplazar el check de `permissions.SUPPORT.PARTNER.key` por checks separados para `PARTNER_ADMIN` y `PARTNER_CONSULTANT`
- Definir prioridad: PARTNER_ADMIN antes que PARTNER_CONSULTANT en la cadena de if/else

### 3.3 Actualizar routing (PrivateRoute)
**Archivo:** `src/managers/RouterManager/index.jsx`

- Cambiar el wrapper de partner routes:
  ```jsx
  <PrivateRoute roles={[AUTH_USERS.PARTNER_ADMIN, AUTH_USERS.PARTNER_CONSULTANT]} module={MODULES_CODE.PARTNER}>
  ```
- Las rutas internas que solo debe ver PARTNER_ADMIN (subscriptions, access) necesitan un PrivateRoute adicional:
  ```jsx
  <PrivateRoute roles={[AUTH_USERS.PARTNER_ADMIN]}>
  ```

### 3.4 Actualizar SubMenuPartner
**Archivo:** `src/modules/partner/managerData/menues/SubMenuPartner/index.jsx`

- Usar `useSecurity()` o `useSession()` para detectar si el usuario es PARTNER_ADMIN o PARTNER_CONSULTANT
- PARTNER_CONSULTANT: solo mostrar item "Organizaciones"
- PARTNER_ADMIN: mostrar "Organizaciones", "Subscripciones", "Acceso" (y Access Points si tiene módulo)

### 3.5 Actualizar PartnerAccessPage (columna nueva)
**Archivo:** `src/modules/partner/managerData/pages/PartnerAccessPage/components/PartnerAccessTable/hooks/usePartnerAccessTableConfig.js`

- Agregar nueva columna con TaggedSwitch para PARTNER_CONSULTANT
- Renombrar la columna actual de PARTNER a "Partner Admin"
- La nueva columna "Consultor" permite activar/desactivar el permiso PARTNER_CONSULTANT por usuario

**Archivo:** `src/modules/partner/managerData/pages/PartnerAccessPage/components/PartnerAccessTable/index.jsx`
- Agregar handler `handleToggleConsultant` similar a `handleTogglePartner`
- Actualizar `hasPartnerPermission` para diferenciar entre PARTNER_ADMIN y PARTNER_CONSULTANT
- Regla de negocio: un usuario NO puede tener PARTNER_ADMIN y PARTNER_CONSULTANT simultáneamente (mutuamente excluyentes, confirmar con el usuario)

**Archivo:** `src/modules/partner/services/mutation/useTogglePartnerPermission.js`
- Actualizar la mutation para aceptar `permissionType` como argumento adicional

### 3.6 Actualizar ManagePartnerProxyUsersModal (modal de organizaciones)
**Archivo:** `src/modules/partner/managerData/pages/OrganizationsPage/components/ManagePartnerProxyUsersModal/index.jsx`

- Detectar el rol del usuario actual (PARTNER_ADMIN o PARTNER_CONSULTANT)
- **Si es PARTNER_ADMIN**: el modal muestra los usuarios con permiso PARTNER_ADMIN y su toggle de proxy (columna "Administrado", comportamiento actual)
- **Si es PARTNER_CONSULTANT**: el modal muestra los usuarios con permiso PARTNER_CONSULTANT y su toggle de asignación (columna "Consultor Partner")
- Actualizar la query `useGetPartnerOrganizationUsersWithPartnerRole` para pasar el filtro de rol

### 3.7 Actualizar contextos de notificaciones
**Archivos:**
- `src/modules/core/contexts/NewNotificationContexts/useManagerAgentSupportMessages.jsx`
- `src/modules/core/contexts/NewNotificationContexts/useManagerNotificationNewMessages.jsx`

- Cambiar `user?.activeRole === ROLES.PARTNER` por `user?.activeRole === ROLES.PARTNER_ADMIN || user?.activeRole === ROLES.PARTNER_CONSULTANT`

### 3.8 Traducciones
- Agregar keys de traducción para los nuevos roles y columnas en los archivos i18n del módulo partner

---

## FASE 4: Validaciones y Edge Cases

- Un usuario no debería poder tener PARTNER_ADMIN y PARTNER_CONSULTANT al mismo tiempo (confirmar)
- Al quitar PARTNER_ADMIN a un usuario, si tiene mapeos proxy, no eliminarlos automáticamente (el mapeo persiste)
- Al quitar PARTNER_CONSULTANT, los mapeos proxy a orgs se mantienen pero el usuario pierde acceso al módulo partner
- loginPartner y logoutPartner deben funcionar para ambos roles
- El PARTNER_CONSULTANT no puede ejecutar mutations administrativas (crear org, sync proxy users, toggle permissions, resend email)

---

## Orden de Ejecución Sugerido
1. Migración de BD (FASE 1)
2. Constantes y permisos del backend (FASE 2.1, 2.2)
3. Resolvers y servicios del backend (FASE 2.3 - 2.8)
4. Constantes y routing del frontend (FASE 3.1 - 3.3)
5. Menú y páginas del frontend (FASE 3.4 - 3.7)
6. Traducciones (FASE 3.8)
7. Testing integral
