# HighLevel API - Guía de Autenticación OAuth 2.0

Una guía completa para obtener tokens de acceso de la API de HighLevel usando OAuth 2.0.

## 📋 Requisitos Previos

- Cuenta HighLevel activa (de pago)
- Permisos de administrador
- Conocimientos básicos de APIs
- Servidor web para callbacks (usar ngrok para desarrollo local)

## 🔐 Métodos de Autenticación

### OAuth 2.0 (Recomendado)
- ✅ Más seguro
- ✅ Acceso granular por scopes
- ✅ Tokens renovables
- ❌ Más complejo de implementar

### Private Integrations
- ✅ Implementación simple
- ❌ Solo para uso interno

## 🚀 Proceso Paso a Paso

### 1. Registrar Aplicación

1. Ve a https://marketplace.gohighlevel.com/
2. Accede a "My Apps" o "Developer"
3. Crea nueva aplicación:
   - Nombre y descripción
   - Redirect URI: `https://tu-dominio.com/auth/callback`
   - Selecciona scopes necesarios

4. Guarda las credenciales:
   ```
   CLIENT_ID=tu_client_id
   CLIENT_SECRET=tu_client_secret
   ```

### 2. Scopes Comunes

```bash
# Contactos
contacts.readonly          # Leer contactos
contacts.write            # Crear/editar contactos

# Conversaciones
conversations.readonly     # Leer conversaciones
conversations.write       # Enviar mensajes

# Calendarios
calendars.readonly        # Leer calendarios
calendars.write          # Modificar calendarios

# Ubicaciones
locations.readonly        # Leer información locations
```

### 3. URL de Autorización

Construye la URL:
```
https://marketplace.gohighlevel.com/oauth/chooselocation?response_type=code&client_id=TU_CLIENT_ID&redirect_uri=TU_REDIRECT_URI&scope=contacts.readonly contacts.write&state=RANDOM_STRING
```

### 4. Intercambiar Código por Token

Petición POST a `https://services.leadconnectorhq.com/oauth/token`:

```bash
curl -X POST https://services.leadconnectorhq.com/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=TU_CLIENT_ID" \
  -d "client_secret=TU_CLIENT_SECRET" \
  -d "grant_type=authorization_code" \
  -d "code=CODIGO_RECIBIDO" \
  -d "redirect_uri=TU_REDIRECT_URI"
```

### 5. Respuesta del Token

```json
{
  "access_token": "eyJ0eXAiOiJKV1Qi...",
  "token_type": "Bearer",
  "expires_in": 86400,
  "refresh_token": "def50200abc123...",
  "scope": "contacts.readonly contacts.write"
}
```

## 🔄 Renovar Token

Cuando el token expire (24h), usa el refresh_token:

```bash
curl -X POST https://services.leadconnectorhq.com/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=TU_CLIENT_ID" \
  -d "client_secret=TU_CLIENT_SECRET" \
  -d "grant_type=refresh_token" \
  -d "refresh_token=TU_REFRESH_TOKEN"
```

## 🛠️ Usar el Token

Headers para todas las peticiones API:
```
Authorization: Bearer tu_access_token
Version: 2021-07-28
Content-Type: application/json
```

Ejemplo - Obtener contactos:
```bash
curl -X GET "https://services.leadconnectorhq.com/contacts/" \
  -H "Authorization: Bearer tu_access_token" \
  -H "Version: 2021-07-28"
```

## ⚠️ Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `invalid_client` | CLIENT_ID/SECRET incorrectos | Verificar credenciales |
| `invalid_grant` | Código expirado (10 min) | Generar nuevo código |
| `redirect_uri_mismatch` | URL no coincide | Verificar URL exacta |
| `insufficient_scope` | Faltan permisos | Solicitar scopes necesarios |

## 🔒 Mejores Prácticas

### Seguridad
- ✅ Usa variables de entorno para credenciales
- ✅ Encripta tokens en base de datos
- ✅ HTTPS en todas las comunicaciones
- ✅ Valida parámetro `state` (previene CSRF)

### Manejo de Tokens
- ✅ Implementa renovación automática
- ✅ Maneja errores 401 gracefully
- ✅ Implementa rate limiting
- ✅ Logs sin incluir tokens

## 🧪 Testing

### Endpoints de prueba:
```bash
# Verificar token
GET https://services.leadconnectorhq.com/oauth/locationToken

# Info de location
GET https://services.leadconnectorhq.com/locations/{location_id}
```

### Herramientas:
- **ngrok**: Exponer servidor local
- **Postman**: Testing de APIs
- **Browser DevTools**: Debuggear redirects

## 📝 Ejemplo Básico (Python)

```python
import requests

# Configuración
CLIENT_ID = "tu_client_id"
CLIENT_SECRET = "tu_client_secret"
REDIRECT_URI = "tu_redirect_uri"

# Intercambiar código por token
def get_access_token(auth_code):
    data = {
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
        'grant_type': 'authorization_code',
        'code': auth_code,
        'redirect_uri': REDIRECT_URI
    }
    
    response = requests.post(
        'https://services.leadconnectorhq.com/oauth/token',
        data=data,
        headers={'Content-Type': 'application/x-www-form-urlencoded'}
    )
    
    return response.json()

# Usar API
def get_contacts(access_token):
    headers = {
        'Authorization': f'Bearer {access_token}',
        'Version': '2021-07-28'
    }
    
    response = requests.get(
        'https://services.leadconnectorhq.com/contacts/',
        headers=headers
    )
    
    return response.json()
```

## 📚 Recursos

- [Documentación Oficial](https://highlevel.stoplight.io/docs/integrations)
- [Developer Marketplace](https://marketplace.gohighlevel.com/)
- [Support Portal](https://help.gohighlevel.com/)

## 📋 Checklist de Implementación

- [ ] Cuenta HighLevel configurada
- [ ] Aplicación creada en marketplace
- [ ] Credenciales guardadas de forma segura
- [ ] URL de callback configurada
- [ ] Scopes seleccionados correctamente
- [ ] Flujo OAuth implementado
- [ ] Renovación de tokens automática
- [ ] Manejo de errores implementado
- [ ] Testing completo realizado

## 🚨 Notas Importantes

- Los tokens expiran cada **24 horas**
- Los códigos de autorización expiran en **10 minutos**
- Las APIs V1 serán deprecadas el **30 septiembre 2025**
- Siempre usar la versión más reciente (V2)
- Implementar rate limiting (respeta límites de HighLevel)

---

**⚠️ Advertencia:** Nunca expongas tus credenciales CLIENT_SECRET en código público o frontend. Úsalas solo en tu backend seguro.
