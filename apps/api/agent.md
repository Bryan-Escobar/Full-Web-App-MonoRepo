# Agent Guide - Planify

## 1) Propósito de Planify
Planify es un sistema universitario para construir y publicar horarios académicos con apoyo automático, manteniendo control operativo por rol y trazabilidad del proceso por ciclo.

Problema que resuelve:
- Coordinar materias, secciones, sesiones, disponibilidad docente, aulas y tiempos en un solo flujo.
- Reducir conflictos de traslape y carga operativa al generar horarios.
- Permitir intervención manual para resolver excepciones antes de publicar.

Resultado esperado del sistema:
- Un horario oficial publicado para cada ciclo académico.
- Consulta interna por rol (administrativo/docente).
- Vista pública final de oferta académica para alumnos/visitantes.

Este documento describe la visión objetivo funcional del sistema para que un agente pueda entender el propósito de cada módulo y de cada tabla, aunque la implementación total aún esté en progreso.

## 2) Roles y responsabilidades

### ADMINISTRADOR GENERAL
- Tiene visibilidad global de todas las facultades.
- Gestiona configuración institucional (estructura y gobierno general).
- Puede observar horarios generados de toda la universidad.
- Puede publicar horarios (global o por alcance habilitado).

### ADMINISTRADOR DE FACULTAD
- Opera con scoping de su facultad.
- Configura insumos académicos de su alcance (materias/secciones/sesiones/aulas/docentes vinculados).
- Ejecuta generación automática de horarios.
- Gestiona excepciones manuales (reasignaciones y resolución de conflictos).
- Publica horarios de su alcance.

### DOCENTE
- Visualiza catálogo de materias de sus facultades vinculadas.
- Selecciona materias para impartir.
- Gestiona disponibilidad/preferencias horarias.
- Revisa y confirma (o rechaza) su carga en el módulo de carga académica.
- Consulta horario oficial consolidado una vez publicado.

### VISITANTE
- Consulta oferta académica pública publicada (fase final del producto).
- Filtra y revisa horarios sin autenticación.

## 3) Flujo operativo del sistema (end-to-end)
1. Configuración académica base:
- Se define estructura institucional (facultad, carrera, edificios, aulas, tipo de aula).
- Se crean materias, secciones y sesiones para el ciclo.

2. Preparación del ciclo:
- Se registra el ciclo académico activo.
- Se define oferta y parámetros de sesiones (duración, tipo de aula, cupos).

3. Participación docente:
- Docente marca disponibilidad y preferencias de horario.
- Docente selecciona materias/secciones.
- Regla funcional vigente: selección aprobada inmediata con posibilidad de revocación por Admin Facultad.

4. Generación automática:
- Admin de Facultad ejecuta la generación para su alcance.
- El sistema asigna timeslots y aulas según insumos académicos y reglas.
- Si no todo puede asignarse, se produce reporte/listado de excepciones.

5. Resolución de excepciones:
- Admin de Facultad ajusta manualmente clases no asignadas o con conflicto.
- Puede mover clase a otro timeslot y/o aula disponible.

6. Confirmación docente de carga:
- En "Mi Carga Académica", cada docente valida su carga resultante antes de publicación.

7. Publicación:
- Admin Facultad o Admin General publica horarios.
- A partir de publicación, el horario se considera oficial para consulta.

8. Consulta pública (fase final):
- El sistema expone la oferta académica publicada para alumnos/visitantes.

## 4) Catálogo funcional de tablas (núcleo)

### 4.1 `facultad`
- Propósito: catálogo de facultades académicas.
- Relaciones clave: se relaciona con `carrera`, `edificio` y vinculaciones de usuario en `usuario_facultad`.
- Reglas de negocio: cada registro representa una unidad organizativa que define alcance administrativo.
- Módulos/roles: gestión institucional (Admin General), scoping operativo (Admin Facultad), filtros de oferta pública.

### 4.2 `carrera`
- Propósito: define carreras/planes de formación dentro de una facultad.
- Relaciones clave: FK a `facultad`; se relaciona con `materia`.
- Reglas de negocio: organiza la oferta académica por programa.
- Módulos/roles: configuración académica (Admin Facultad/Admin General), consulta docente y pública.

### 4.3 `tipo_aula`
- Propósito: clasifica aulas por tipo (teórica, práctica, cómputo, etc.).
- Relaciones clave: referenciada por `aula` y por `sesion`.
- Reglas de negocio: una sesión debe asignarse a un aula compatible con su tipo requerido.
- Módulos/roles: configuración de recursos y reglas de asignación (Admins).

### 4.4 `edificio`
- Propósito: agrupa físicamente aulas y las asocia a facultad.
- Relaciones clave: FK a `facultad`; relación 1:N con `aula`.
- Reglas de negocio: permite búsqueda de aulas por ubicación/facultad.
- Módulos/roles: gestión de infraestructura (Admin Facultad/Admin General), resolución de excepciones.

### 4.5 `aula`
- Propósito: inventario de espacios físicos disponibles para asignación.
- Relaciones clave: FK a `tipo_aula` y `edificio`; utilizada por `asignacion`.
- Reglas de negocio: cada aula aporta capacidad máxima y compatibilidad de uso.
- Módulos/roles: administración de aulas, motor de generación, ajuste manual.

### 4.6 `ciclo_academico`
- Propósito: delimita temporalmente la planificación y publicación de horarios.
- Relaciones clave: referenciado por `seccion`, `asignacion`, `preferencia_materia`.
- Reglas de negocio: solo un ciclo activo a la vez; su estado gobierna acciones permitidas.
- Módulos/roles: administración de ciclo, generación, publicación, consulta.

### 4.7 `usuario`
- Propósito: perfil funcional interno del usuario autenticado.
- Relaciones clave: FK a `auth.users`; relacionado con `usuario_facultad`; referenciado como docente en tablas académicas.
- Reglas de negocio: define rol y atributos operativos (nombre, carga, especialidad).
- Módulos/roles: autenticación/autorización, perfil, scoping y flujos docentes.

### 4.8 `usuario_facultad`
- Propósito: tabla puente de pertenencia usuario-facultad (multi-facultad).
- Relaciones clave: FK a `usuario` y `facultad`.
- Reglas de negocio: habilita docentes o admins en más de una facultad.
- Módulos/roles: control de alcance, catálogo docente, navegación por facultades.

### 4.9 `materia`
- Propósito: catálogo académico de asignaturas ofertables.
- Relaciones clave: FK a `carrera`; relación con `seccion`.
- Reglas de negocio: la materia define base académica de secciones y sesiones.
- Módulos/roles: creación de oferta (Admins), selección docente, consulta pública.

### 4.10 `seccion`
- Propósito: instancia de apertura de una materia en un ciclo.
- Relaciones clave: FK a `materia` y `ciclo_academico`; relación con `sesion`.
- Reglas de negocio: controla cupo y partición operativa (incluyendo subsecciones).
- Módulos/roles: gestión de oferta, generación de horario, resolución de excepciones.

### 4.11 `sesion`
- Propósito: unidad programable de una sección (bloques, cupo esperado, tipo de aula requerido).
- Relaciones clave: FK a `seccion` y `tipo_aula`; relación con `asignacion`.
- Reglas de negocio: una sección puede tener múltiples sesiones; cada sesión es la entidad asignable a horario/aula.
- Módulos/roles: generación automática, validación de compatibilidad, ajuste manual.

### 4.12 `timeslot`
- Propósito: catálogo de bloques de tiempo (día + franja horaria).
- Relaciones clave: referenciado por `asignacion` y `disponibilidad_docente`.
- Reglas de negocio: estandariza la rejilla semanal para disponibilidad y asignación.
- Módulos/roles: calendario de disponibilidad, motor de asignación, visualización de malla.

### 4.13 `asignacion`
- Propósito: resultado de programar una sesión en un ciclo, timeslot y aula.
- Relaciones clave: FK a `sesion`, `ciclo_academico`, `timeslot`, `aula`.
- Reglas de negocio: representa el horario operativo/resultado de generación y ajustes manuales.
- Módulos/roles: generación automática, excepciones, publicación, consulta docente/pública.

### 4.14 `disponibilidad_docente`
- Propósito: registro de disponibilidad y prioridad horaria por docente.
- Relaciones clave: FK a `timeslot` (y docente vía campo de referencia del modelo vigente).
- Reglas de negocio: alimenta la lógica de asignación para respetar preferencias laborales.
- Módulos/roles: "Mis Preferencias" (Docente), motor de generación, revisión administrativa.

### 4.15 `preferencia_materia`
- Propósito: captura relación de preferencia/selección del docente sobre secciones del ciclo.
- Relaciones clave: FK a `seccion` y `ciclo_academico` (y docente vía campo de referencia del modelo vigente).
- Reglas de negocio: soporta priorización de carga y trazabilidad de interés/selección.
- Módulos/roles: catálogo docente, aprobación/revocación administrativa, preparación de generación.

## 5) Matriz Rol x Módulo x Tablas

| Módulo funcional | Admin General | Admin Facultad | Docente | Visitante | Tablas principales |
|---|---|---|---|---|---|
| Gestión institucional | Crear/Editar/Consultar | Consultar | No | No | `facultad`, `carrera`, `tipo_aula`, `edificio`, `aula` |
| Gestión de ciclo | Crear/Editar/Publicar/Consultar | Crear/Editar/Publicar/Consultar (scope) | Consultar | No | `ciclo_academico` |
| Catálogo académico | Crear/Editar/Consultar | Crear/Editar/Consultar (scope) | Consultar | Consultar (publicado) | `materia`, `seccion`, `sesion`, `carrera` |
| Gestión de usuarios y facultades | Crear/Editar/Consultar | Consultar (scope) | Consultar propio perfil | No | `usuario`, `usuario_facultad` |
| Preferencias docente | Consultar | Consultar (scope) | Crear/Editar/Consultar | No | `disponibilidad_docente`, `preferencia_materia`, `timeslot` |
| Generación automática | Consultar resultado global | Ejecutar/Consultar | No | No | `seccion`, `sesion`, `timeslot`, `aula`, `asignacion` |
| Excepciones manuales | Consultar | Resolver/Editar | No | No | `asignacion`, `seccion`, `sesion`, `aula`, `timeslot` |
| Mi Carga Académica | Consultar | Consultar (scope) | Confirmar/Rechazar/Consultar | No | `asignacion`, `seccion`, `sesion`, `preferencia_materia` (y registro de confirmación según modelo vigente) |
| Publicación de horario | Publicar/Consultar | Publicar/Consultar | Consultar | Consultar | `ciclo_academico`, `asignacion`, `seccion`, `sesion` |
| Oferta pública | Supervisar | Supervisar | No | Consultar/Descargar | `ciclo_academico`, `asignacion`, `materia`, `seccion`, `aula`, `edificio` |

## 6) Máquina de estados del ciclo y del horario

### 6.1 Estados del ciclo académico
Estados funcionales definidos para operar el proceso:
- `BORRADOR`: ciclo en configuración, aún sin propuesta consolidada.
- `EN_GENERACION`: ejecución de generación automática en curso.
- `GENERADO`: propuesta generada, pendiente de ajustes y/o publicación.
- `PUBLICADO`: horario oficial vigente para consulta operativa y pública.
- `CERRADO`: ciclo finalizado; se conserva para historial.

### 6.2 Transiciones funcionales
- `BORRADOR -> EN_GENERACION`: al iniciar generación automática.
- `EN_GENERACION -> GENERADO`: al finalizar asignación automática.
- `GENERADO -> EN_GENERACION`: regeneración automática desde cero cuando se vuelve a ejecutar.
- `GENERADO -> PUBLICADO`: publicación oficial.
- `PUBLICADO -> CERRADO`: cierre formal del ciclo.

### 6.3 Estado funcional del horario (vista operativa)
El horario puede observarse como:
- Propuesto (recién generado),
- Con excepciones (requiere intervención manual),
- Ajustado (sin conflictos pendientes),
- Oficial (publicado).

Estos estados operativos se reflejan mediante datos de `asignacion`, excepciones detectadas y estado del ciclo.

## 7) Módulos frontend objetivo

### 7.1 Gestión de Ciclos
- Objetivo: crear, activar, navegar y administrar el ciclo de trabajo.
- Entradas: año, período, estado del ciclo.
- Salidas: ciclo seleccionado como contexto global.
- Tablas: `ciclo_academico`.

### 7.2 Gestión Académica (Materias, Secciones, Sesiones)
- Objetivo: construir la oferta de cada ciclo.
- Entradas: materia, carrera, cupos, duración, tipo de sesión.
- Salidas: oferta lista para asignación.
- Tablas: `materia`, `seccion`, `sesion`, `carrera`, `tipo_aula`.

### 7.3 Gestión de Recursos (Edificios y Aulas)
- Objetivo: mantener infraestructura asignable.
- Entradas: edificio, aula, capacidad, tipo.
- Salidas: recursos disponibles para motor y ajustes manuales.
- Tablas: `edificio`, `aula`, `tipo_aula`, `facultad`.

### 7.4 Gestión de Usuarios y Alcance
- Objetivo: controlar acceso por rol y facultad.
- Entradas: rol, vínculos de facultad, datos de perfil.
- Salidas: permisos y scoping operativo.
- Tablas: `usuario`, `usuario_facultad`, `facultad`.

### 7.5 Catálogo Docente y Postulación/Selección
- Objetivo: que docente gestione materias objetivo de su carga.
- Entradas: filtros por facultad/carrera/materia, selección docente.
- Salidas: preferencias/selecciones registradas para planificación.
- Tablas: `preferencia_materia`, `materia`, `seccion`, `ciclo_academico`.

### 7.6 Mis Preferencias (Disponibilidad)
- Objetivo: registrar disponibilidad semanal por timeslot.
- Entradas: día/bloque, prioridad, estado.
- Salidas: matriz de disponibilidad utilizable por el motor.
- Tablas: `disponibilidad_docente`, `timeslot`.

### 7.7 Generación Automática
- Objetivo: producir propuesta de horario por facultad/ciclo.
- Entradas: secciones, sesiones, aulas, disponibilidad, timeslots.
- Salidas: asignaciones + listado de excepciones.
- Tablas: `seccion`, `sesion`, `aula`, `timeslot`, `asignacion`.

### 7.8 Excepciones de Horario
- Objetivo: resolver casos no asignados o conflictivos.
- Entradas: sección/sesión con problema, alternativas de aula/timeslot/docente.
- Salidas: horario ajustado y consistente.
- Tablas: `asignacion`, `seccion`, `sesion`, `aula`, `timeslot`.

### 7.9 Mi Carga Académica (HDU-35)
- Objetivo: permitir al docente aprobar/rechazar su carga asignada antes de publicación.
- Entradas: asignaciones del docente y decisión de confirmación.
- Salidas: estado de aceptación de carga por docente/ciclo.
- Tablas: `asignacion`, `seccion`, `sesion` y entidad de confirmación definida en el modelo SQL vigente.

### 7.10 Publicación de Horarios
- Objetivo: oficializar el horario para consumo institucional y externo.
- Entradas: ciclo con asignaciones listas.
- Salidas: horario oficial publicado.
- Tablas: `ciclo_academico`, `asignacion`, `seccion`, `sesion`.

### 7.11 Oferta Académica Pública (fase final)
- Objetivo: exponer horarios publicados para alumnos/visitantes.
- Entradas: filtros de consulta (facultad, carrera, ciclo, materia).
- Salidas: visualización/descarga de oferta oficial.
- Tablas: `ciclo_academico`, `asignacion`, `materia`, `seccion`, `aula`, `edificio`, `carrera`, `facultad`.

## 8) Reglas de negocio transversales
- Scoping por facultad: Admin Facultad opera solo su alcance.
- Visibilidad global: Admin General observa toda la operación.
- Multi-facultad por usuario: controlado por `usuario_facultad`.
- Unicidad de ciclo activo: el sistema trabaja con un contexto vigente único.
- Generación automática ejecutable por Admin Facultad.
- Regeneración automática: al re-ejecutar, reconstruye desde cero la propuesta del ciclo.
- Excepciones: se resuelven manualmente antes de publicar.
- Publicación: habilitada para Admin Facultad y Admin General.
- Consulta pública: solo sobre información ya publicada.
- Preferencias docentes: se registran y se consideran en la planificación automática.

## 9) Glosario funcional
- Ciclo académico: período lectivo sobre el que se planifica y publica horario.
- Materia: asignatura ofertable de una carrera.
- Sección: apertura de una materia en un ciclo.
- Sesión: unidad programable de una sección (bloques/condiciones de aula).
- Timeslot: bloque temporal estándar (día y franja horaria).
- Asignación: resultado de ubicar una sesión en aula y timeslot.
- Excepción: caso no asignado o conflictivo que requiere intervención manual.
- Carga académica: conjunto de secciones/sesiones asignadas a un docente.
- Publicación: acción de oficializar el horario del ciclo.
- Oferta pública: vista externa de horarios publicados para alumnos/visitantes.

## 10) Criterios de validación de este documento
- Existe en la ruta `@/Planify/agent.md`.
- Está redactado en español.
- Incluye visión objetivo del sistema (no estado actual de implementación).
- Describe propósito, relaciones y reglas por tabla.
- Incluye flujos end-to-end, matriz rol/módulo/tablas y estados del ciclo/horario.
- Incluye explícitamente el módulo `Mi Carga Académica` (HDU-35).
- Incluye oferta pública como fase final del producto.
- Agrega/actualiza reglas al documento si son modificadas en el futuro

## 11) Anexo operativo backend (API actual)
Esta sección documenta el estado operativo de endpoints implementados en backend para facilitar onboarding técnico, sin reemplazar la visión funcional anterior.

### 11.1 Módulo Usuario (`/api/usuarios`)
- Propósito operativo: CRUD de perfiles internos (`usuario`) consumidos por autenticación/autorización y módulos académicos.
- Endpoints:
  - `GET /api/usuarios`
  - `GET /api/usuarios/:id`
  - `POST /api/usuarios`
  - `PUT /api/usuarios/:id`
  - `DELETE /api/usuarios/:id`
- Reglas operativas:
  - `usuario_id` se valida como UUID.
  - Respuesta estándar: `ApiResponse`.
  - Manejo de errores centralizado: `handleError` + `CustomError`.
  - Relación de alcance por facultad se gestiona en `usuario_facultad`.

### 11.2 Módulo Usuario-Facultad (`/api/usuario-facultades`)
- Propósito operativo: administrar pertenencia multi-facultad de usuarios.
- Endpoints:
  - `GET /api/usuario-facultades`
  - `GET /api/usuario-facultades/:id`
  - `GET /api/usuario-facultades/usuario/:usuarioId`
  - `GET /api/usuario-facultades/facultad/:facultadId`
  - `POST /api/usuario-facultades`
  - `DELETE /api/usuario-facultades/:id`
- Reglas operativas:
  - Valida existencia de `usuario` y `facultad` al crear.
  - Evita relaciones duplicadas (`409`).

### 11.3 Módulo Edificios (`/api/edificios`)
- Propósito operativo: CRUD de edificios asociados a facultad.
- Endpoints:
  - `GET /api/edificios`
  - `GET /api/edificios/:id`
  - `GET /api/edificios/facultad/:facultadId`
  - `POST /api/edificios`
  - `PUT /api/edificios/:id`
  - `DELETE /api/edificios/:id`
- Reglas operativas:
  - Valida `facultad_id` existente en create/update.
  - Evita duplicidad de código de edificio dentro de una facultad (`409`).
  - Bloquea eliminación si tiene aulas asociadas (`409`).

### 11.4 Módulo Tipos de Aula (`/api/tipos-aula`)
- Propósito operativo: CRUD del catálogo `tipo_aula`.
- Endpoints:
  - `GET /api/tipos-aula`
  - `GET /api/tipos-aula/:id`
  - `POST /api/tipos-aula`
  - `PUT /api/tipos-aula/:id`
  - `DELETE /api/tipos-aula/:id`
- Reglas operativas:
  - Evita duplicidad por nombre de tipo (`409`).
  - Bloquea eliminación si existen `aula` o `sesion` dependientes (`409`).

### 11.5 Módulo Aulas (`/api/aulas`)
- Propósito operativo: CRUD de inventario de aulas y filtros por contexto organizativo.
- Endpoints:
  - `GET /api/aulas`
  - `GET /api/aulas/:id`
  - `GET /api/aulas/edificio/:edificioId`
  - `GET /api/aulas/tipo-aula/:tipoAulaId`
  - `GET /api/aulas/facultad/:facultadId`
  - `POST /api/aulas`
  - `PUT /api/aulas/:id`
  - `DELETE /api/aulas/:id`
- Reglas operativas:
  - Valida existencia de `edificio` y `tipo_aula` en create/update.
  - Evita duplicidad de código de aula dentro del mismo edificio (`409`).
  - Bloquea eliminación si hay asignaciones asociadas (`409`).
