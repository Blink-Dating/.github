## 🧹 Resumen de Módulos

Esta documentación desglosa los componentes clave de la infraestructura técnica de Blink. Cada módulo está diseñado para reflejar los valores fundamentales de Blink: seguridad, respeto y conexión significativa.

### Módulos cubiertos

* [x] Autenticación
* [ ] Perfil de Usuario
* [ ] Motor de Emparejamiento
* [ ] Flujo de Conversación
* [ ] Incorporación
* [ ] Moderación y Reportes
* [ ] Notificaciones
* [ ] Panel de Administración y Métricas
* [ ] Infraestructura y Despliegue

---

# 🔐 Módulo de Autenticación

Blink utiliza un **servidor propio** como proveedor de autenticación centralizada, eliminando la dependencia directa de Supabase en el flujo de cliente. El servidor integra una capa personalizada de seguridad y control sobre las credenciales, los tokens y las políticas de acceso.

### Principios clave

* Mínima fricción para el usuario
* Gestión robusta y segura de tokens (`access_token` + `refresh_token`)
* Lógica unificada en el servidor para todos los flujos críticos
* Cumplimiento con la filosofía de seguridad primero de Blink

---

### 🔄 Flujo de Inicio de Sesión

* **Método**: API REST del servidor
* **Gestión de tokens**: El servidor emite `access_token` (JWT) + `refresh_token` firmado y seguro.
* **Persistencia de sesión**: Gestión en el cliente usando `refresh_token` para renovar el `access_token`.

**Flujo:**

1. El usuario introduce correo electrónico y contraseña en la aplicación.
2. La aplicación llama a `POST /api/v1/auth/login`.
3. El servidor valida las credenciales y genera `access_token` + `refresh_token`.
4. Los tokens se almacenan de forma segura en el cliente (cookies HttpOnly o almacenamiento cifrado).
5. El `refresh_token` permite renovar automáticamente el `access_token` antes de que expire.

---

### 🌙 Flujo de Renovación de Token

* **Método**: API REST del servidor
* **Punto de acceso**: `POST /api/v1/auth/refresh`

**Flujo:**

1. El cliente detecta que el `access_token` está caducado o próximo a caducar.
2. El cliente llama a `POST /api/v1/auth/refresh` enviando el `refresh_token`.
3. El servidor valida el `refresh_token` y emite un nuevo par `access_token` + `refresh_token`.
4. El cliente actualiza los tokens de forma segura.

---

### 🌟 Flujo de Registro

* **Método**: API REST del servidor
* **Punto de acceso**: `POST /api/v1/auth/users`

**Ejemplo de petición:**

```http
POST /api/v1/auth/users
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password",
  "termsAndConditions": {
    "accepted": true,
    "version": "1.0"
  }
}
```

**Acciones en el servidor:**

* Valida correo electrónico y contraseña.
* Crea el usuario en la base de datos.
* Guarda metadatos personalizados (nombre, preferencias, etc.).
* Envía correo electrónico de confirmación si está habilitado.

---

### 📉 Verificación mediante Código de un Solo Uso (OTP)

* **Flujo**: OTP por correo electrónico gestionado desde el servidor.
* **Interfaz del cliente**: Recoge el OTP e invoca la API para verificarlo.
* **Servidor**: Verifica OTP y completa el inicio de sesión o registro.

**Ventajas:**

* Permite inicio de sesión sin contraseña.
* Puede usarse como verificación de identidad o segundo factor de autenticación.
* Permite personalizar completamente el flujo de OTP y su apariencia.

---

### 🔁 Gestión de Contraseñas

#### Contraseña Olvidada

* **Punto de acceso**: `POST /api/v1/auth/forgot-password`
* **Acción**: Envía correo electrónico para restablecer la contraseña.

```http
POST /api/v1/auth/forgot-password
Content-Type: application/json

{
  "email": "user@example.com"
}
```

#### Cambio de Contraseña

* **Punto de acceso**: `POST /api/v1/auth/change-password`
* **Requiere autenticación**: Sí (usuario autenticado)

```http
POST /api/v1/auth/change-password
Content-Type: application/json

{
  "currentPassword": "old_pass",
  "newPassword": "new_pass"
}
```

---

### 🔐 Consideraciones de Seguridad

* ✅ Confirmación de correo electrónico obligatoria
* ✅ Políticas de seguridad a nivel de filas (RLS) habilitadas en la base de datos (si corresponde)
* ✅ Limitación de frecuencia en todos los puntos de acceso sensibles
* ✅ Validación completa de los datos de entrada en el servidor
* ✅ Segundo factor de autenticación opcional en la hoja de ruta
* ✅ `refresh_token` firmado y seguro, con posibilidad de revocación
* ✅ Los tokens se almacenan en cookies HttpOnly o en almacenamiento cifrado

---

### 🧪 Pruebas y Validaciones

| Flujo                    | Punto de Acceso  | Estado |
| ------------------------ | ---------------- | ------ |
| Inicio de sesión         | API del servidor | ✅      |
| Registro                 | API del servidor | ✅      |
| Inicio de sesión con OTP | API del servidor | ✅      |
| Contraseña olvidada      | API del servidor | ✅      |
| Cambio de contraseña     | API del servidor | ✅      |
| Renovación de token      | API del servidor | ✅      |

---

### 📌 Pendiente / Mejoras

* [ ] Añadir inicio de sesión mediante proveedores externos (Google, Apple)
* [ ] Integrar segundo factor de autenticación opcional (TOTP)
* [ ] Mejorar la experiencia de usuario en la renovación de sesión en móvil
* [ ] Herramientas para administradores: restablecer contraseña / desactivar usuario

---

### 📄 Versionado de Términos y Política de Privacidad

Las versiones de los Términos y Condiciones y de la Política de Privacidad mostradas a los usuarios están codificadas en la aplicación móvil. Esto garantiza la coherencia entre lo que se muestra y lo que se almacena durante el registro o las actualizaciones de consentimiento.

* No existe un **punto de acceso de la API** para recuperar estas versiones de forma dinámica.
* Cualquier cambio en los documentos legales debe ir acompañado de una actualización de la aplicación que refleje los nuevos números de versión y las URL correspondientes.

Este enfoque garantiza que la lógica de la aplicación y el cumplimiento legal estén acoplados y sean predecibles en cada versión.

---

## 🛠 Notas para el Equipo de Desarrollo

* Mantener la coherencia entre las validaciones del cliente y las del servidor
* Implementar la monitorización en todos los puntos de acceso de autenticación (métricas y alertas)
* Mantener el sistema del servidor y sus dependencias actualizadas por seguridad
* Nunca exponer la lógica de emisión o validación de tokens en el cliente

---
