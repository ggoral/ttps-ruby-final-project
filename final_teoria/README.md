# Trabajo final de la materia

El alumno podrá optar el trabajo final a realizar entre dos propuestas
realizadas por la cátedra o
realizar una sugerencia que será evaluada por el profesor.
Básicamente, sea cual sea el trabajo, la idea es que el sistema cumpla los
siguientes puntos:

  * El sistema interactúe con otras aplicaciones mediante servicios: *en lo
    posible la [API de
recursos](https://github.com/TTPS-ruby/sandbox/tree/master/trabajo_final)*
  * Se desarrolle utilizando Rails
  * Se aplique TDD: considerando tests de unidad y de funcionales

## Opción 1: desarrollar un plugin para [Redmine](http://www.redmine.org)

### Enunciado

El plugin debe proveer un módulo para gestión de recursos basado en la [API
desarrollada](https://github.com/TTPS-ruby/sandbox/tree/master/trabajo_final).

El plugin deberá:

  * Permitir la configuración de la URL de la API globalmente en redmine
  * A los proyectos, en su configuración se habilitará la opción de habilitar el
    módulo de recursos que provee el plugin
  * En los proyectos que tengan habilitado el módulo, deberá mostrarse en el
    menú del proyecto, un acceso a recursos, donde podrán listarse los recursos
disponibles
    * Para cada recurso podrán visualizarse todas las reservas permitiendo
      filtrar por estado 
    * Ver la disponibilidad de un recurso
    * Reservar un recurso
    * Autorizar un recurso: lo podrá hacer un rol específico
    * Cancelar un recurso: lo podrá hacer un rol específico o el mismo usuario
      que haya realizado la reserva
  * Debe permitir asociar a un ticket, un recurso de la API
  * Recibir notificaciones por mail cuando:
    * Una reserva cambia el estado
    * Un tiempo X en horas configurable en el plugin, antes que una reserva
      aprobada se cumpla
  


### Consideraciones

Los recursos de la API no se asocian a una entidad, por lo tanto los recursos
serán comunes a todos los proyectos redmine. Si desea puede modificar la API
agregando a los recursos un campo entity al recurso que podría usarse en este
caso como id de proyecto. Entonces, la API debería agregar un servicio:

  * `GET /resources_for_entity/:entity` que devolvería los recursos para una
    entidad específica

Sería conveniente agregar en la API un servicio de agregado y modificación de
recursos:

  * `POST /resources` crea un nuevo recurso
  * `PUT /resources/:id` actualiza los datos de un recurso

### Links de interés

* [Guía para el desarrollador de
  Redmine](http://www.redmine.org/projects/redmine/wiki/Developer_Guide)
* [Desarrollo de plugins en
  redmine](http://www.redmine.org/projects/redmine/wiki/Plugin_Tutorial)

## Opción 2: desarrollar una aplicación Rails para Gestión de Aulas

### Enunciado

La aplicación deberá:

   * Permitir la reserva de aulas por docentes, pero la aprobación de los
     pedidos la realizará un rol diferente que llamaremos Celador

   * Permitir al celador agregar nuevas aulas

   * Los usuarios deberán autenticarse usando sus cuentas de Google, Twitter y/o
     Facebook

   * La gestión de aulas (recursos), se realizará por medio de la [API
de recursos](https://github.com/TTPS-ruby/sandbox/tree/master/trabajo_final)

   * Para cada aula, un docente/celador podrá:
     * Listar horarios disponibles
     * Listar las reservas permitiendo filtrar por estado 
     * Reservar el aula
     * Cancelar las reservas propias
   * El celador podrá autorizar las reservas pendientes o cancelarlas
   * Recibir notificaciones por mail cuando:
     * Una reserva cambia el estado
     * Un tiempo X en horas configurable, antes que una reserva aprobada se
       cumpla


### Consideraciones

Sería conveniente agregar en la API un servicio de agregado y modificación de
recursos:

  * `POST /resources` crea un nuevo recurso
  * `PUT /resources/:id` actualiza los datos de un recurso

