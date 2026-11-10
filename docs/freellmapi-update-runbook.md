# FreeLLMAPI Gateway — actualización segura

Runbook operativo para futuras actualizaciones del add-on FreeLLMAPI Gateway administrado por Home Assistant Supervisor.

> **Objetivo:** actualizar la versión de FreeLLMAPI sin perder datos persistentes ni interrumpir innecesariamente el cliente OpenCode que depende de este servicio.
>
> **Alcance:** este procedimiento corresponde a la instalación local de FreeLLMAPI Gateway como add-on/app de Home Assistant. No asumir que FreeLLMAPI se ejecuta en un notebook Windows, ni sustituir el mecanismo de Supervisor por Docker Compose sin comprobar primero quién administra el contenedor.

## Principios no negociables

- Supervisor debe gestionar la instalación/reconstrucción del add-on.
- No ejecutar `docker compose pull`, `docker compose up -d`, `docker build` ni recrear manualmente el contenedor salvo que se demuestre que esta instalación dejó de estar administrada por Supervisor.
- No reiniciar manualmente Home Assistant Core ni el contenedor.
- No borrar ni recrear volúmenes. La persistencia de la aplicación debe conservarse (en esta instalación se identificó `/app/server/data`; confirmar el montaje real antes de cualquier operación).
- No modificar OpenCode, su provider, endpoint, API key ni modelo predeterminado como parte de esta actualización.
- No guardar tokens, API keys, archivos de entorno, dumps ni datos privados en Git.
- No declarar éxito solo porque se modificaron `Dockerfile` y `config.yaml`, o porque el puerto responde. La versión instalada y una inferencia real deben quedar verificadas.

## 0. Identificar la instalación y el mecanismo real

Antes de escribir:

1. Localizar y leer completo el script existente `update_freellmapi.sh`.
2. Inspeccionar `/local_apps/freellmapi_gateway/Dockerfile`, `config.yaml`, README, backups y logs relevantes.
3. Confirmar que la entidad existe: `update.freellmapi_gateway_update`.
4. Confirmar que el add-on/app es administrado por Supervisor y que el mecanismo de actualización de esta instalación utiliza el servicio de Home Assistant `update.install`.
5. Confirmar las rutas reales, la imagen/tag, el almacenamiento persistente y el método de autenticación. No asumir que un token de Supervisor y un token de Home Assistant son intercambiables.

Si el mecanismo no coincide con este runbook, detenerse y documentar la diferencia antes de ejecutar cambios.

## 1. Consultar versión instalada

La fuente operativa de versión instalada es el atributo `installed_version` de la entidad de actualización.

Ejemplo REST (reemplazar la URL si Home Assistant no escucha en localhost):

```bash
curl -fsS "http://127.0.0.1:8123/api/states/update.freellmapi_gateway_update" \
  -H "Authorization: Bearer ${HA_TOKEN}" \
  -H "Content-Type: application/json" |
  jq '{state, installed_version: .attributes.installed_version, latest_version: .attributes.latest_version, in_progress: .attributes.in_progress}'
```

La ruta correcta del atributo en jq es `.attributes.installed_version` (no `.attributesinstalled_version`).

Usar un token de acceso de Home Assistant con permisos suficientes. No escribir el token literalmente en scripts versionados, documentación pública, argumentos visibles ni historial de shell.

## 2. Consultar última release upstream

Repositorio upstream:

- Releases: https://github.com/tashfeenahmed/freellmapi/releases
- API latest: https://api.github.com/repos/tashfeenahmed/freellmapi/releases/latest

Consulta:

```bash
curl -fsS https://api.github.com/repos/tashfeenahmed/freellmapi/releases/latest |
  jq '{tag_name, published_at, html_url}'
```

Verificar que se trate de una release estable y revisar sus notas. Normalizar el tag quitando el prefijo `v` al comparar con la versión del add-on, cuando corresponda.

Comparar:

- versión instalada real (`installed_version`);
- versión publicada en GitHub;
- versión declarada en `config.yaml`;
- tag de imagen en `Dockerfile`.

No asumir que las cuatro fuentes están sincronizadas.

## 3. Preparar el cambio

Antes de modificar:

1. Crear backups fechados de, como mínimo, `Dockerfile` y `config.yaml`.
2. Registrar versión instalada, versión objetivo, tag de imagen actual y estado de la entidad.
3. Confirmar dónde está montado el almacenamiento persistente y que no se va a borrar.
4. Revisar notas de release por migraciones o cambios incompatibles.
5. Mostrar el diff exacto y obtener confirmación explícita antes de aplicar una actualización real.

En el procedimiento conocido, los cambios de versión se limitan a:

- tag de imagen en `Dockerfile`, por ejemplo `ghcr.io/tashfeenahmed/freellmapi:vX.Y.Z`;
- campo `version` en `config.yaml`.

Modificar únicamente esos valores; preservar el resto del contenido.

## 4. Solicitar la actualización a Supervisor

En esta instalación, el mecanismo previsto es el servicio de Home Assistant:

- Servicio: `update.install`
- Entidad: `update.freellmapi_gateway_update`

REST:

```bash
curl -fsS -X POST "http://127.0.0.1:8123/api/services/update/install" \
  -H "Authorization: Bearer ${HA_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"entity_id":"update.freellmapi_gateway_update"}'
```

También puede invocarse mediante Home Assistant MCP / UI:

- Domain: `update`
- Service: `install`
- Target entity: `update.freellmapi_gateway_update`

**Importante:** ejecutar `update.install` solo cuando Home Assistant reconozca una actualización disponible. Si responde `No update available`, no hacer rollback de los archivos para intentar forzar la detección. Investigar primero por qué `latest_version` no refleja la versión esperada: catálogo/repositorio del add-on, manifiesto, caché/recarga del catálogo o mecanismo de publicación local. No improvisar un rebuild alternativo.

## 5. Monitorear el proceso

Consultar periódicamente el estado de la entidad mediante REST y esperar a que:

- `in_progress` sea `false`;
- `installed_version` coincida con la versión objetivo;
- el estado final sea coherente (habitualmente `off` cuando no queda actualización pendiente).

Ejemplo de consulta:

```bash
curl -fsS "http://127.0.0.1:8123/api/states/update.freellmapi_gateway_update" \
  -H "Authorization: Bearer ${HA_TOKEN}" |
  jq '{state, installed_version: .attributes.installed_version, latest_version: .attributes.latest_version, in_progress: .attributes.in_progress}'
```

No considerar la actualización terminada por el mero hecho de que el servicio haya aceptado la petición. Si no progresa, inspeccionar el estado y logs del add-on/Supervisor; no encadenar reinicios.

## 6. Validación posterior

### 6.1 Versión instalada

Volver a consultar la entidad y confirmar que `installed_version` es exactamente la versión objetivo y `in_progress=false`.

### 6.2 API y catálogo de modelos

Ejemplo (usar la URL real de la instalación y una API key suministrada de forma segura):

```bash
curl -fsS "http://HOST_DEL_GATEWAY:3001/v1/models" \
  -H "Authorization: Bearer ${FREELLMAPI_KEY}" |
  jq '.data | length'
```

No imprimir ni registrar el valor de la API key.

### 6.3 Inferencia real con Auto

```bash
curl -fsS -X POST "http://HOST_DEL_GATEWAY:3001/v1/chat/completions" \
  -H "Authorization: Bearer ${FREELLMAPI_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"model":"auto","messages":[{"role":"user","content":"Respondé únicamente: OK"}],"max_tokens":8}'
```

Confirmar que la respuesta contiene una salida generada y no solamente un HTTP 200 vacío o un mensaje de error del router.

### 6.4 Dashboard

Abrir el dashboard de la instalación. En el entorno documentado durante noviembre de 2026 se utilizó una dirección LAN privada en el puerto 3001; no asumir que esa dirección es válida en otros despliegues.

### 6.5 Persistencia y cliente

- Confirmar que los datos persistentes siguen presentes y que el montaje no cambió.
- Confirmar que OpenCode sigue usando su provider habitual y que una consulta real a través de `freellmapi/auto` funciona.
- No cambiar ni reiniciar OpenCode como parte del procedimiento.

## 7. Si algo falla

- **`No update available`:** no revertir `Dockerfile` ni `config.yaml` para engañar al updater. Comparar `installed_version`, `latest_version`, manifiesto y catálogo del repositorio/app; revisar cómo se refresca el catálogo en esta instalación.
- **Error de autenticación:** comprobar el método y token utilizados por el script original. No pegar tokens en el chat, Git ni logs.
- **Update en progreso indefinidamente:** revisar logs y estado antes de tomar otra acción. No disparar múltiples instalaciones concurrentes.
- **API no responde después del update:** revisar logs y estado del add-on; conservar backups y datos. No borrar volúmenes ni reiniciar Core como primera medida.
- **Rollback:** solo si existe una causa concreta y un procedimiento probado para volver a la imagen anterior, preservando el almacenamiento persistente. Registrar qué se revirtió y validar API e inferencia.

## 8. Registro de cada actualización

Guardar en el registro operativo (sin secretos):

| Campo | Valor |
|---|---|
| Fecha/hora y zona | |
| Versión instalada antes | |
| Release upstream objetivo | |
| Tag de imagen | |
| Diff aplicado | |
| Backup de configuración | |
| Resultado de `update.install` | |
| Versión instalada después | |
| API /v1/models | PASS / FAIL |
| Inferencia `auto` | PASS / FAIL |
| Dashboard | PASS / FAIL |
| Persistencia | PASS / FAIL |
| Estado final | PASS / PARTIAL / NO-GO |

**Criterio de éxito:** la versión objetivo aparece como instalada en Home Assistant y FreeLLMAPI completa una inferencia real con `auto`, manteniendo los datos persistentes y el cliente OpenCode operativo.
