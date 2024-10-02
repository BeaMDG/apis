# Integración de API B con API A

Este documento describe el flujo de integración entre la API B y la API A. API B consume datos de facturación de la API A, los transforma y los devuelve en su propio formato. Además, se incluye la documentación de las APIs y un diagrama del flujo de trabajo.

## Aclaraciones

A considerar el hecho de que es más eficiente realizar la transformación de los datos directamente en la API B, en lugar de utilizar un componente intermedio, ya que simplifica el flujo, lo que reduciría la latencia y los puntos de fallo. La API B ya poseería la lógica de negocio y es el punto final que entiende el formato requerido. También es más fácil de mantener y escalar ya que cualquier cambio que se realice en la API A solo implicaría realizar otro cambio en la API B y no en otro componente más.

## Objetivo
El objetivo principal de esta integración es obtener las facturas desde la API A en un rango de fechas determinado y enviar estos datos a la API B para su posterior procesamiento.

## Índice

1. [Flujo de Integración](#flujo-de-integración)
2. [Documentación de la API A](#documentación-de-la-api-a)
3. [Documentación de la API B](#documentación-de-la-api-b)
4. [Diagrama de Flujo](#diagrama-de-flujo)
5. [Integración entre API A y API B](#integración-entre-api-a-y-api-b)

## Flujo requerido

El flujo de integración entre la API B y la API A sigue estos pasos:

## Flujo Requerido

El flujo de integración entre la API B y la API A sigue estos pasos:

1. **Solicitud del Cliente a la API B**:

   - El Cliente envía una solicitud `GET` a la API B con los parámetros `start_date` y `end_date`.
     
   - **Respuesta esperada**:
     - La API B devuelve una respuesta solicitando la clave API B.

3. **Cliente envía clave API a API B**:
   
   - El Cliente responde con la clave API en el encabezado.
    
   - **Códigos de Estado HTTP esperados**:
     - `200 OK`: Autenticación aceptada y la solicitud es procesada.
     - `401 Unauthorized`: Clave API inválida.

5. **API B obtiene token de autenticación de API A**:
   - API B realiza una solicitud `POST` a la API A para obtener el token de acceso
     
   - **Códigos de Estado HTTP esperados**:
     - `200 OK`: Token de acceso recibido correctamente.
     - `400 Bad Request`: Credenciales incorrectas.
     - `401 Unauthorized`: Credenciales de cliente inválidas.

6. **API B solicita facturas a API A**:
   - Con el token de acceso recibido, API B realiza una solicitud `GET` a la API A para obtener las facturas.
    
   - **Códigos de Estado HTTP esperados**:
     - `200 OK`: Facturas obtenidas correctamente.
     - `400 Bad Request`: Parámetros inválidos en la solicitud.
     - `401 Unauthorized`: Token no válido.
     - `404 Not Found`: No se encontraron facturas para el rango de fechas solicitado.
     - `500 Internal Server Error`: Error en el servidor de API A.

7. **Transformación de Datos en API B**:
   - API B transforma los datos recibidos de la API A al formato esperado por sus consumidores. La transformación de los datos incluye renombrar los campos de la siguiente manera:

| Campo API A   | Campo API B    |
|---------------|----------------|
| `id`          | `invoice_id`   |
| `cliente`     | `customer`     |
| `monto`       | `amount_due`   |
| `fecha_emision`| `date_issued` |

8. **API B responde al Cliente**:
   - API B responde al Cliente con las facturas transformadas en el formato de API B.


## Documentación de la API A

Aqui encontrarás la documentación de la API A [Documentación API A](https://github.com/BeaMDG/apis/blob/main/documentaci%C3%B3n-API_A.yaml)

###Obtención del Token de Autenticación API A

Para acceder a la API A, es necesario obtener un token de autenticación. A continuación, se describen los pasos necesarios para obtener dicho token.

1. **Credenciales de Cliente**:
   - Asegúrate de tener tu `client_id` y `client_secret`. Estas son proporcionadas por el proveedor de la API.

2. **Realizar la Solicitud POST**:
   - Ejecuta el siguiente comando `curl` para obtener el token:

   ```bash
   curl -X POST https://api.sistemaA.com/oauth/token \
   -H "Content-Type: application/x-www-form-urlencoded" \
   -d "grant_type=client_credentials&client_id=TU_CLIENT_ID&client_secret=TU_CLIENT_SECRET"```

 3. **Ejemplo de respuesta**:<br>
    {
  "access_token": "ACCESS_TOKEN",
  "token_type": "bearer",
  "expires_in": 3600
  }
   
## Documentación de la API B
Aqui encontrarás la documentación de la API B [Documentacion API B](https://github.com/BeaMDG/apis/blob/main/documentaci%C3%B3n-API_B.yaml)

## Diagrama de Flujo

A continuación se muestra el diagrama de flujo que describe la interacción entre **Cliente** <<>> **API B** <<>> **API A**.

![Diagrama de Flujo](https://github.com/BeaMDG/apis/blob/main/diagrama_api.png)

# Integración entre API A y API B

## Introducción
La integración entre la API A (Sistema de Facturación A) y la API B (Sistema de Facturación B) permite obtener y transformar datos de facturación. La API A se utiliza para consultar facturas, mientras que la API B se encarga de recibir estos datos y devolverlos en un formato específico.

## Flujo de Integración
1. **Obtener Token de Autenticación de la API A:**
   - Realizar una solicitud POST a la URL `https://api.sistemaA.com/oauth/token` con las credenciales de cliente para obtener un token.

2. **Consulta de Facturas a la API A:**
   - Usar el token obtenido para realizar una solicitud GET al endpoint de la API A (`/facturas`) especificando las fechas requeridas.

3. **Transformación de Datos:**
   - Procesar la respuesta de la API A para adaptarla al formato requerido por la API B. Esto puede incluir renombrar campos y cambiar tipos de datos.

4. **Enviar Datos a la API B:**
   - Realizar una solicitud GET al endpoint de la API B (`/bills`) con los datos transformados y la clave API en el encabezado.

5. **Manejo de Respuestas:**
   - Procesar la respuesta de la API B, que confirmará si los datos fueron recibidos correctamente.




