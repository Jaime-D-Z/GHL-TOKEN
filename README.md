# Cómo Obtener el Token de Acceso de HighLevel API

Este README te guiará paso a paso para principiantes puedan obtener un token de acceso para la API de HighLevel utilizando OAuth 2.0.

## Requisitos Previos

- Una cuenta de HighLevel (GoHighLevel)
- Acceso al Developer Marketplace de HighLevel
- Una aplicación registrada en el marketplace de desarrolladores

## Métodos de Autenticación

HighLevel ofrece diferentes métodos de autenticación:

### 1. OAuth 2.0 (Recomendado para aplicaciones públicas)
Para aplicaciones que necesitan acceso en nombre de múltiples usuarios o locations.

### 2. Private Integrations (Para uso interno)
Para aplicaciones internas que solo acceden a datos de tu propia agencia.

### 3. API Keys (Método legacy)
Método más simple pero con limitaciones de seguridad.

## Proceso OAuth 2.0 - Paso a Paso

### Paso 1: Registrar tu Aplicación

1. Accede al [HighLevel Developer Marketplace](https://marketplace.gohighlevel.com/)
2. Ve a la sección "My Apps" o "Mis Aplicaciones"
3. Crea una nueva aplicación con los siguientes datos:
   - **Nombre de la aplicación**
   - **Descripción**
   - **Redirect URI** (URL donde recibirás el código de autorización)
   - **Scopes/Permisos** requeridos

4. Guarda los siguientes valores generados:
   - `CLIENT_ID`
   - `CLIENT_SECRET`

### Paso 2: Obtener Código de Autorización

Construye la URL de autorización:

```
https://marketplace.gohighlevel.com/oauth/chooselocation?response_type=code&redirect_uri={REDIRECT_URI}&client_id={CLIENT_ID}&scope={SCOPES}
```

**Parámetros:**
- `response_type`: Siempre debe ser "code"
- `redirect_uri`: Tu URL de redirección (debe coincidir con la registrada)
- `client_id`: El CLIENT_ID de tu aplicación
- `scope`: Los permisos que necesitas (separados por espacios)

**Ejemplo de scopes comunes:**
- `contacts.readonly` - Leer contactos
- `contacts.write` - Escribir contactos
- `conversations.readonly` - Leer conversaciones
- `calendars.readonly` - Leer calendarios
- `locations.readonly` - Leer información de locations

### Paso 3: Intercambiar Código por Token

Una vez que el usuario autorice tu aplicación, será redirigido a tu `redirect_uri` con un código de autorización:

```
https://tu-redirect-uri.com/callback?code=AUTHORIZATION_CODE&location_id=LOCATION_ID
```

Realiza una petición POST para obtener el token de acceso:

```bash
curl -X POST https://services.leadconnectorhq.com/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Accept: application/json" \
  -d "client_id=TU_CLIENT_ID" \
  -d "client_secret=TU_CLIENT_SECRET" \
  -d "grant_type=authorization_code" \
  -d "code=AUTHORIZATION_CODE" \
  -d "redirect_uri=TU_REDIRECT_URI"
```

### Paso 4: Respuesta del Token

La respuesta será similar a:

```json
{
  "access_token": "tu_access_token_aqui",
  "token_type": "Bearer",
  "expires_in": 86400,
  "refresh_token": "tu_refresh_token_aqui",
  "scope": "contacts.readonly conversations.readonly"
}
```

**Importante:** El access token es válido solo por 24 horas.

## Renovar el Token de Acceso

Cuando el token expire, usa el `refresh_token` para obtener uno nuevo:

```bash
curl -X POST https://services.leadconnectorhq.com/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Accept: application/json" \
  -d "client_id=TU_CLIENT_ID" \
  -d "client_secret=TU_CLIENT_SECRET" \
  -d "grant_type=refresh_token" \
  -d "refresh_token=TU_REFRESH_TOKEN"
```

## Usar el Token de Acceso

Incluye el token en el header de tus peticiones API:

```bash
curl -X GET "https://services.leadconnectorhq.com/contacts/" \
  -H "Authorization: Bearer tu_access_token_aqui" \
  -H "Version: 2021-07-28"
```

## Ejemplo en Python

```python
import requests
import json

# Configuración
CLIENT_ID = "tu_client_id"
CLIENT_SECRET = "tu_client_secret"
REDIRECT_URI = "tu_redirect_uri"
AUTHORIZATION_CODE = "codigo_obtenido"

# Obtener token de acceso
token_url = "https://services.leadconnectorhq.com/oauth/token"

token_data = {
    "client_id": CLIENT_ID,
    "client_secret": CLIENT_SECRET,
    "grant_type": "authorization_code",
    "code": AUTHORIZATION_CODE,
    "redirect_uri": REDIRECT_URI
}

token_response = requests.post(token_url, data=token_data)
token_info = token_response.json()

access_token = token_info["access_token"]
refresh_token = token_info["refresh_token"]

# Usar el token para hacer peticiones
headers = {
    "Authorization": f"Bearer {access_token}",
    "Version": "2021-07-28",
    "Content-Type": "application/json"
}

response = requests.get("https://services.leadconnectorhq.com/contacts/", headers=headers)
print(response.json())
```

## Ejemplo en JavaScript/Node.js

```javascript
const axios = require('axios');

// Configuración
const CLIENT_ID = 'tu_client_id';
const CLIENT_SECRET = 'tu_client_secret';
const REDIRECT_URI = 'tu_redirect_uri';
const AUTHORIZATION_CODE = 'codigo_obtenido';

// Función para obtener token de acceso
async function getAccessToken() {
    try {
        const tokenResponse = await axios.post('https://services.leadconnectorhq.com/oauth/token', {
            client_id: CLIENT_ID,
            client_secret: CLIENT_SECRET,
            grant_type: 'authorization_code',
            code: AUTHORIZATION_CODE,
            redirect_uri: REDIRECT_URI
        }, {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        });

        return tokenResponse.data;
    } catch (error) {
        console.error('Error obteniendo token:', error.response.data);
    }
}

// Función para usar el token
async function makeApiCall(accessToken) {
    try {
        const response = await axios.get('https://services.leadconnectorhq.com/contacts/', {
            headers: {
                'Authorization': `Bearer ${accessToken}`,
                'Version': '2021-07-28'
            }
        });

        return response.data;
    } catch (error) {
        console.error('Error en la llamada API:', error.response.data);
    }
}
```

## Mejores Prácticas

1. **Almacenamiento Seguro**: Nunca expongas tus credenciales en el código del lado del cliente
2. **Renovación Automática**: Implementa un sistema para renovar tokens automáticamente antes de que expiren
3. **Manejo de Errores**: Implementa manejo de errores para tokens expirados (HTTP 401)
4. **Scopes Mínimos**: Solo solicita los permisos que realmente necesitas
5. **HTTPS**: Siempre usa HTTPS para todas las comunicaciones

## Troubleshooting

### Error: "invalid_client"
- Verifica que el CLIENT_ID y CLIENT_SECRET sean correctos
- Asegúrate de que la aplicación esté activa en el marketplace

### Error: "invalid_grant"
- El código de autorización puede haber expirado (válido solo por 10 minutos)
- Verifica que el redirect_uri coincida exactamente

### Error: "insufficient_scope"
- Tu token no tiene los permisos necesarios para el endpoint
- Solicita los scopes adicionales durante la autorización

## Enlaces Útiles

- [Documentación Oficial de HighLevel API](https://highlevel.stoplight.io/docs/integrations)
- [Developer Marketplace](https://marketplace.gohighlevel.com/)
- [Support Portal](https://help.gohighlevel.com/)

## Notas Importantes

- Las APIs V1 serán deprecadas el 30 de septiembre de 2025
- Siempre usa la versión más reciente de la API (V2)
- Los tokens de acceso expiran cada 24 horas y deben ser renovados

---
