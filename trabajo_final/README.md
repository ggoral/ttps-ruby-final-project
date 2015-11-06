# API de administración de recursos

La organización X cuenta con una gran cantidad de recursos que administra.
Entre estos recursos podemos encontrar objetos como Computadoras o espacios
físicos como Oficinas o Salas.

Lo que se quiere hacer es un mecanismo que permita a los usuarios de esa
organización ocupar recursos. Una solicitud de ocupación de un recurso consta
de:

* Recurso a ocupar
* Fecha y hora de inicio de ocupación
* Fecha y hora de fin de ocupación (no obligatoria, debido a que la ocupación
puede ser indefinida)
* Usuario que ocupará el recurso

Cuando se solicita una ocupación, debe chequearse que el recurso esté
disponible, en cuyo caso la reserva quedará pendiente de aprobación. Si no
estuviera disponible deberá indicarse la no disponibilidad mediante un código
de error.

Luego de realizada la solicitud, la misma puede ser aprobada o rechazada.
En caso de aceptarse el recurso pasa a estar ocupado, y en caso de rechazarse,
el recurso sigue estando disponible.

Una ocupación puede cancelarse en cualquier momento: estando pendiente,
asignada o incluso en uso.

## Servicios de la API

### Recursos

#### Listar todos los recursos

`GET /resources HTTP/1.1`

Devuelve estado `200 OK` y el siguiente `body`:

```json
{
  "resources": [
    {
      "name": "Computadora",
      "description": "Notebook con 4GB de RAM y 256 GB de espacio en disco con Linux",
      "links": [
        {
          "rel": "self",
          "uri": "http://localhost:9292/resources/1"
        }
      ]
    },
    {
      "name": "Monitor",
      "description": "Monitor de 24 pulgadas SAMSUNG",
      "links": [
        {
          "rel": "self",
          "uri": "http://localhost:9292/resources/2"
        }
      ]
    },
    {
      "name": "Sala de reuniones",
      "description": "Sala de reuniones con máquinas y proyector",
      "links": [
        {
          "rel": "self",
          "uri": "http://localhost:9292/resources/3"
        }
      ]
    },
  ],
  "links": [
    {
      "rel": "self",
      "uri": "http://localhost:9292/resources"
    }
  ]
}
```

#### Ver un recurso

`GET /resources/1 HTTP/1.1`

Devuelve estado `200 OK` y el siguiente `body`:

```json
{
  "resource": {
    "name": "Computadora",
    "description": "Notebook con 4GB de RAM y 256 GB de espacio en disco con Linux",
    "links": [
      {
        "rel": "self",
        "uri": "http://localhost:9292/resources/1"
      },
      {
        "rel": "bookings",
        "uri": "http://localhost:9292/resources/1/bookings"
      }
    ]
  }
}
```

## Asignación de recursos

### Listar reservas de un recurso

Lista los pedidos de reserva autorizados para un recurso a partir de una fecha 

`GET /resources/1/bookings?date='YYYY-MM-DD&limit=30&status=all' HTTP/1.1`

#### Argumentos

* date: fecha a partir de la cuál se debe verificar la disponibilidad. Si no
se especifica se asume la fecha de mañana.
* limit: cantidad de días para los cuales considerar la búsqueda. Si no se
especifica se asume 30. Este valor no podrá ser mayor que 365.
* status: pending|approved|all por defecto se asume approved.

Devuelve estado `200 OK` y el siguiente `body`:

```json
{
  "bookings": [
    {
      "start": "2013-10-26T10:00:00Z",
      "end": "2013-10-26T11:00:00Z",
      "status": "approved",
      "user": "someuser@gmail.com",
      "links": [
        {
          "rel": "self",
          "uri": "http://localhost:9292/resources/1/bookings/100"
        },
        {
          "rel": "resource",
          "uri": "http://localhost:9292/resources/1"
        },
        {
          "rel": "accept",
          "uri": "http://localhost:9292/resource/1/bookings/100",
          "method": "PUT"
        },
        {
          "rel": "reject",
          "uri": "http://localhost:9292/resource/1/bookings/100",
          "method": "DELETE"
        }
      ]
    },
    { 
      "start": "2013-10-26T11:00:00Z",
      "end": "2013-10-26T12:30:00Z",
      "status": "approved",
      "user": "otheruser@gmail.com",
      "links": [
          {
            "rel": "self",
            "uri": "http://localhost:9292/resources/1/bookings/101"
          },
          {
            "rel": "resource",
            "uri": "http://localhost:9292/resources/1"
          },          
          {
            "rel": "accept",
            "uri": "http://localhost:9292/resource/1/bookings/101",
            "method": "PUT"
          },
          {
            "rel": "reject",
            "uri": "http://localhost:9292/resource/1/bookings/101",
            "method": "DELETE"
          } 
       ]
    }
  ],
  "links": [
    {
      "rel": "self",
      "uri": "http://localhost:9292/resources/1/bookings?date=2013-10-26&limit=1&status=approved"
    }
  ]
}
```

Si el recurso solicitado no existe deberá devolver 404.

### Disponibilidad de un recurso a partir de una fecha

Lista los bloques de tiempo disponibles para reservar el recurso.

`GET /resources/1/availability?date=YYYY-MM-DD&limit=30 HTTP/1.1`

Para calcular los bloques disponibles de tiempo se consideran las reservas
canceladas o inexistentes en el sistema. Por ejemplo, si asumimos que para el
recurso 2 tenemos las siguientes reservas:

 * Aceptada una reserva para el día 13/11/2013 de 11:00 a 12:00 
 * Pendiente una reserva para el día 13/11/2013 de 14:00 a 15:00
 * Aceptada una reserva para el día 14/11/2013 de 11:00 a 12:00

Un pedido de `/resources/1/availability?date=2013-11-12&limit=3` debería devolver:

* Disponibilidad desde 12/11/2013 00:00:00 hasta 13/11/2013 11:00:00
* Disponibilidad desde 13/11/2013 12:00:00 hasta 14/11/2013 11:00:00
* Disponibilidad desde 14/11/2013 12:00:00 hasta 15/11/2013 00:00:00

Devuelve estado `200 OK` y el siguiente `body`:

```json
{
  "availability": [
    {
      "from": "2013-11-12T00:00:00Z",
      "to": "2013-11-13T11:00:00Z",
      "links": [
        {
          "rel": "book",
          "link": "http://localhost:9292/resources/1/bookings",
          "method": "POST"
        },
        {
          "rel": "resource",
          "uri": "http://localhost:9292/resources/1"
        },
      ]
    },
    {
      "from": "2013-11-13T12:00:00Z",
      "to": "2013-11-14T11:00:00Z",
      "links": [
        {
          "rel": "book",
          "link": "http://localhost:9292/resources/1/bookings",
          "method": "POST"
        },
        {
          "rel": "resource",
          "uri": "http://localhost:9292/resources/1"
        }        
      ]
    },
    {
      "from": "2013-11-14T12:00:00Z",
      "to": "2013-11-15T00:00:00Z",
      "links": [
        {
          "rel": "book",
          "link": "http://localhost:9292/resources/1/bookings",
          "method": "POST"
        },
        {
          "rel": "resource",
          "uri": "http://localhost:9292/resources/1"
        },        
      ]
    }
  ],
  "links": [
    {
      "rel": "self"
      "link": "http://localhost:9292/resources/1/availability?date=2013-11-12&limit=3"
    }
  ]
}
```
 
### Reservar recurso

Reserva un recurso en el bloque de tiempo indicado

`POST /resources/1/bookings HTTP/1.1`

#### Argumentos

Los argumentos son parte del cuerpo del pedido HTTP POST y deben indicar:

 * from: fecha y hora de inicio de la reserva
 * to: fecha y hora de fin de la reserva

Devuelve estado `201 CREATED` y el siguiente `body`:

```json
{
  "book":
  {
    "from": "2013-11-12T00:00:00Z",
    "to": "2013-11-13T11:00:00Z",
    "status": "pending",
    "links": [
      {
        "rel": "self",
        "url": "http://localhost:9292/resources/1/bookings/#ID_CREADO#"
      },
      {
        "rel": "accept",
        "uri": "http://localhost:9292/resource/1/bookings/#ID_CREADO#",
        "method": "PUT"
      },
      {
        "rel": "reject",
        "uri": "http://localhost:9292/resource/1/bookings/#ID_CREADO#",
        "method": "DELETE"
      }
    ]
  }
}
```	

En caso que el bloque solicitado esté ocupado debrá retornar código de error
`409 CONFLICT`.

### Cancelar reserva

Cancela una reserva existente.

`DELETE /resources/1/bookings/100 HTTP/1.1`

En caso de una eliminación satisfactoria debera responder con estado `200 OK` y el cuerpo de la respuesta vacío.

En caso que la reserva indicada no exista deberá retornar un código de error `404 NOT FOUND`


### Autorizar reserva

Autoriza una reserva pendiente. En caso de que haya otras reservas para el mismo recurso pendientes, todas son canceladas.

`PUT /resources/1/bookings/100 HTTP/1.1`

En caso de una autorización satisfactoria debera responder con estado `200 OK` y el el siguiente `body`: 

```json
{
  "book":
  {
    "from": "2013-11-12T00:00:00Z",
    "to": "2013-11-13T11:00:00Z",
    "status": "apporved",
    "links": [
      {
        "rel": "self",
        "url": "http://localhost:9292/resources/1/bookings/100"
      },
      {
        "rel": "accept",
        "uri": "http://localhost:9292/resource/1/bookings/100",
        "method": "PUT"
      },
      {
        "rel": "reject",
        "uri": "http://localhost:9292/resource/1/bookings/100",
        "method": "DELETE"
      },
      {
        "rel": "resource",
        "url": "http://localhost:9292/resources/1"
      }      
    ]
  }
}
```

En caso de que la reserva indicada no exista deberá retornar un código de error `404 NOT FOUND`
En caso de que ya exista una reserva aprobada para dicho lapso deberá retornar un código de error `409 CONFLICT`


### Mostrar una reserva

Visualiza una reserva existente.

`GET /resources/1/bookings/100 HTTP/1.1`

```json
{
  "from": "2013-11-12T00:00:00Z",
  "to": "2013-11-13T11:00:00Z",
  "status": "pending",
  "links": [
    {
      "rel": "self",
      "url": "http://localhost:9292/resources/1/bookings/#ID_CREADO#"
    },
    {
      "rel": "resource",
      "uri": "http://localhost:9292/resource/1",
    },
    {
      "rel": "accept",
      "uri": "http://localhost:9292/resource/1/bookings/#ID_CREADO#",
      "method": "PUT"
    },
    {
      "rel": "reject",
      "uri": "http://localhost:9292/resource/1/bookings/#ID_CREADO#",
      "method": "DELETE"
    }
  ]
}
```

## Notas

* Las fechas deberán manejarse siguiendo el standard [ISO 8601](http://es.wikipedia.org/wiki/ISO_8601)
* La API deberá respetar la arquitectura [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer)
* Los links en los ejemplos utilizan http://localhost:9292 asumiendo que la API
corre en este puerto. Pero si el día de mañana la API se instalace en
http://reservatuscosas.acme.com, todas las URLs de los ejemplos deberán cambiar.
* El trabajo deberá implementarse usando TDD, lo que implica que inicialmente
deberán escribirse los TESTS antes del código de la API.
* El trabajo deberá seguir los coding standards (si no sigue los coding
standards es motivo de desaprobación).
