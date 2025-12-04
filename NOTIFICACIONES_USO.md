# 📬 Sistema de Notificaciones - Guía de Uso

## ✅ Implementación Completada

El sistema de notificaciones ahora es **100% funcional** y se integra automáticamente con:

### 🔔 Notificaciones Automáticas

#### 1. **Mensajes en Chat**
Cuando un usuario envía un mensaje, el receptor recibe automáticamente una notificación con:
- **Título**: "Nuevo mensaje de [Nombre]"
- **Descripción**: Vista previa del mensaje (máx 50 caracteres)
- **Tipo**: `mensaje` 📧
- **Metadata**: chatId, senderId, senderName

```javascript
// ✅ YA IMPLEMENTADO - Se crea automáticamente al enviar mensajes
```

#### 2. **Adición a Grupos**
Para notificar cuando agregas un usuario a un grupo:

```javascript
// Ejemplo desde el dashboard del profesor
await window.notifyGroupAddition(
    userId,           // ID del estudiante agregado
    "Matemáticas 101", // Nombre del grupo
    groupId,          // ID del grupo
    profesorName      // Quien lo agregó
);
```

#### 3. **Logros y Reconocimientos**
Para notificar logros o reconocimientos:

```javascript
await window.notifyAchievement(
    userId,
    "🏆 ¡Primer Logro Desbloqueado!",
    "Completaste tu primera lección de Python"
);
```

### 📊 Características del Sistema

✅ **Notificaciones en Tiempo Real**: Usan Firebase Firestore snapshots
✅ **Badge con Contador**: Muestra número de no leídas
✅ **Marcar como Leída/No leída**: Click en los botones
✅ **Eliminar Notificación**: Con animación suave
✅ **Iconos por Tipo**: 📚 Tareas, ✉️ Mensajes, 🏆 Logros, etc.
✅ **Timestamps Relativos**: "hace 5 minutos", "hace 2 horas"
✅ **Diseño Responsive**: Se adapta a móviles y tablets

### 🎨 Tipos de Notificaciones Disponibles

| Tipo | Icono | Uso |
|------|-------|-----|
| `mensaje` / `message` | ✉️ | Mensajes de chat |
| `tarea` / `task` / `assignment` | 📚 | Tareas asignadas |
| `logro` / `achievement` | 🏆 | Logros desbloqueados |
| `xp` | ⭐ | Puntos de experiencia |
| `nivel` / `level` | 🎯 | Subir de nivel |
| `info` | ℹ️ | Información general |
| `alerta` / `warning` | ⚠️ | Alertas importantes |

### 🔧 Funciones Globales Disponibles

#### `createNotification(userId, type, title, description, metadata)`
Crea cualquier tipo de notificación personalizada.

**Parámetros:**
- `userId` (string): UID del usuario receptor
- `type` (string): Tipo de notificación (ver tabla arriba)
- `title` (string): Título de la notificación
- `description` (string): Descripción/mensaje
- `metadata` (object): Datos adicionales opcionales

**Ejemplo:**
```javascript
await window.createNotification(
    "abc123",
    "tarea",
    "Nueva tarea asignada",
    "Completa los ejercicios de Python básico",
    { taskId: "task123", dueDate: "2025-12-10" }
);
```

#### `notifyGroupAddition(userId, groupName, groupId, addedBy)`
Notifica cuando un usuario es agregado a un grupo.

#### `notifyAchievement(userId, title, description)`
Notifica logros o reconocimientos especiales.

### 📱 Interfaz de Usuario

#### Abrir Panel de Notificaciones:
1. Click en el ícono 🔔 en el header
2. Se muestra el contador de no leídas
3. Modal con todas las notificaciones

#### Acciones Disponibles:
- **Marcar como leída**: ✅ (hover en la notificación)
- **Marcar como no leída**: 📧 (hover en notificación leída)
- **Eliminar**: 🗑️ (con animación de salida)
- **Cerrar panel**: Click en "Cerrar" o fuera del modal

### 🔥 Firestore Structure

```
users/{userId}/notifications/{notificationId}
  ├── type: string
  ├── title: string
  ├── description: string
  ├── message: string (alias de description)
  ├── read: boolean
  ├── metadata: object
  └── createdAt: timestamp
```

### 🚀 Próximas Mejoras Sugeridas

- [ ] Notificaciones push en el navegador
- [ ] Sonido al recibir notificación
- [ ] Filtrar por tipo de notificación
- [ ] Exportar historial de notificaciones
- [ ] Email cuando hay notificaciones importantes

---

## 📝 Notas Importantes

- Las notificaciones se cargan automáticamente al abrir el panel
- El listener se mantiene activo para recibir actualizaciones en tiempo real
- Las notificaciones antiguas se mantienen hasta que el usuario las elimine
- El sistema es compatible con modo oscuro

**¡El sistema está listo para producción!** 🎉
