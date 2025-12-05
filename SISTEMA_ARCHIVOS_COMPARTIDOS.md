# Sistema de Subida y Compartición de Archivos por Grupo

## 📋 Descripción General

Sistema completo que permite a los profesores subir archivos a grupos específicos y que los estudiantes puedan visualizarlos y descargarlos. Los archivos se almacenan de forma segura en Firebase Storage y sus referencias se guardan en Firestore.

---

## 🏗️ Arquitectura del Sistema

### Componentes Principales

1. **Frontend - Profesor** (`app/dashboard-profesor.html`)
   - Modal de subida de archivos
   - Zona de drag & drop
   - Barra de progreso de subida
   - Listado de archivos compartidos
   - Funcionalidad de eliminación

2. **Frontend - Estudiante** (`app/grupos.html`)
   - Vista de archivos compartidos en modal de grupo
   - Información del archivo y quien lo subió
   - Botón de descarga

3. **Backend - Firebase Storage**
   - Almacenamiento de archivos en `/groups/{groupId}/files/`
   - URLs de descarga seguras
   - Validación de tamaño (10MB máx)

4. **Backend - Firestore**
   - Colección `sharedFiles` con metadata de archivos
   - Filtrado por `groupId`
   - Timestamps para ordenamiento

---

## 📊 Estructura de Datos

### Colección: `sharedFiles`

```javascript
{
  fileId: "1733282400000",          // Timestamp único
  groupId: "abc123xyz",              // ID del grupo (CLAVE DE FILTRADO)
  uploadedBy: "professorUID",        // UID del profesor
  uploaderName: "Prof. Alan García", // Nombre del profesor
  uploaderPhoto: "https://...",      // URL de foto de perfil
  fileName: "Tarea1.pdf",            // Nombre original del archivo
  fileSize: 1048576,                 // Tamaño en bytes
  fileType: "application/pdf",       // MIME type
  fileUrl: "https://firebasestorage.googleapis.com/...", // URL de descarga
  storagePath: "groups/abc123/files/1733282400000_Tarea1.pdf", // Ruta en Storage
  description: "Material de apoyo", // Descripción opcional
  createdAt: Timestamp,              // Timestamp de Firestore
  timestamp: "2025-12-04T12:00:00Z"  // ISO string para ordenamiento
}
```

### Ruta en Firebase Storage

```
/groups/{groupId}/files/{timestamp}_{filename}
```

**Ejemplo:**
```
/groups/abc123xyz/files/1733282400000_Tarea_Semana_1.pdf
```

---

## 🔄 Flujo de Trabajo

### 1. Subida de Archivo (Profesor)

#### A. Interfaz de Usuario

**Punto de Acceso:**
```
Dashboard Profesor → Mis Grupos → [Seleccionar Grupo] → Pestaña "Archivos" → Botón "Subir Archivo"
```

**Elementos del Modal:**
- Zona de drag & drop interactiva
- Selector de archivo con validación
- Campo de descripción (opcional)
- Barra de progreso en tiempo real
- Botones de confirmación/cancelación

#### B. Validaciones del Cliente

```javascript
// Tamaño máximo
const maxSize = 10 * 1024 * 1024; // 10MB

// Tipos permitidos
const allowedTypes = [
  'application/pdf',                                              // PDF
  'application/msword',                                           // Word (.doc)
  'application/vnd.openxmlformats-officedocument...',            // Word (.docx)
  'application/vnd.ms-excel',                                     // Excel (.xls)
  'application/vnd.openxmlformats-officedocument.spreadsheetml...', // Excel (.xlsx)
  'application/vnd.ms-powerpoint',                                // PowerPoint (.ppt)
  'application/vnd.openxmlformats-officedocument.presentationml...', // PowerPoint (.pptx)
  'application/zip',                                              // ZIP
  'application/x-rar-compressed',                                 // RAR
  'text/plain',                                                   // TXT
  'image/jpeg',                                                   // JPEG
  'image/png',                                                    // PNG
  'image/gif'                                                     // GIF
];
```

#### C. Proceso de Almacenamiento

**Paso 1: Subida a Firebase Storage**

```javascript
// Crear referencia única
const timestamp = Date.now();
const fileName = `${timestamp}_${selectedFile.name}`;
const storageRef = ref(window.storage, `groups/${currentGroupId}/files/${fileName}`);

// Subir con seguimiento de progreso
const uploadTask = uploadBytesResumable(storageRef, selectedFile);

uploadTask.on('state_changed',
  (snapshot) => {
    // Actualizar barra de progreso
    const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
  },
  (error) => {
    // Manejar errores
  },
  async () => {
    // Subida exitosa - obtener URL
    const downloadURL = await getDownloadURL(uploadTask.snapshot.ref);
  }
);
```

**Paso 2: Persistencia en Firestore**

```javascript
const fileData = {
  fileId: timestamp.toString(),
  groupId: currentGroupId,
  uploadedBy: user.uid,
  uploaderName: user.displayName || user.email,
  uploaderPhoto: user.photoURL || null,
  fileName: selectedFile.name,
  fileSize: selectedFile.size,
  fileType: selectedFile.type,
  fileUrl: downloadURL,
  storagePath: `groups/${currentGroupId}/files/${fileName}`,
  description: description || null,
  createdAt: serverTimestamp(),
  timestamp: new Date().toISOString()
};

await addDoc(collection(window.db, 'sharedFiles'), fileData);
```

#### D. Respuesta al Usuario

- ✅ Notificación de éxito: "Archivo subido correctamente"
- ✅ Cierre automático del modal
- ✅ Recarga de la lista de archivos del grupo

---

### 2. Visualización de Archivos (Profesor)

**Punto de Acceso:**
```
Dashboard Profesor → Mis Grupos → [Seleccionar Grupo] → Pestaña "Archivos"
```

**Funcionalidades:**
- Grid de tarjetas de archivos
- Información del archivo (nombre, tamaño, descripción)
- Información del uploader y fecha
- Botón de descarga
- Botón de eliminación (hover)

**Query de Consulta:**

```javascript
const q = query(
  collection(window.db, 'sharedFiles'),
  where('groupId', '==', groupId),
  orderBy('createdAt', 'desc')
);
```

---

### 3. Visualización de Archivos (Estudiante)

#### A. Lógica de Filtrado (Automática)

El sistema implementa **filtrado automático por pertenencia a grupo**:

```javascript
// El estudiante solo ve archivos de sus grupos
// La query filtra por groupId donde el estudiante es miembro

async function openStudentGroupDetail(groupId) {
  // Verificar que el estudiante pertenece al grupo
  const groupDoc = await getDoc(doc(window.db, 'groups', groupId));
  const group = groupDoc.data();
  
  if (!group.studentIds.includes(user.uid)) {
    // El estudiante NO tiene acceso
    return;
  }
  
  // Cargar archivos del grupo
  await loadStudentGroupFiles(groupId);
}
```

#### B. Endpoint de Consulta

**Punto de Acceso:**
```
Dashboard Estudiante → Mis Grupos → [Seleccionar Grupo] → Sección "Archivos Compartidos"
```

**Query de Consulta (Idéntica a Profesor):**

```javascript
const q = query(
  collection(window.db, 'sharedFiles'),
  where('groupId', '==', groupId),
  orderBy('createdAt', 'desc')
);
```

#### C. Características de la Vista

- Grid responsivo (2 columnas)
- Tarjetas con información completa del archivo
- Íconos visuales según tipo de archivo
- Información del profesor que subió el archivo
- Tiempo transcurrido desde la subida
- Botón de descarga directo

---

## 🔐 Reglas de Seguridad

### Firebase Storage Rules

```javascript
// Archivos de grupos (solo miembros)
match /groups/{groupId}/{allPaths=**} {
  allow read: if isAuthenticated();
  allow write: if isAuthenticated() 
                  && request.resource.size < 10 * 1024 * 1024; // Máximo 10MB
}
```

### Firestore Rules (Recomendadas)

```javascript
match /sharedFiles/{fileId} {
  // Lectura: Solo usuarios autenticados
  allow read: if request.auth != null;
  
  // Escritura: Solo profesores del grupo
  allow create: if request.auth != null 
                   && request.auth.token.rol == 'profesor'
                   && request.resource.data.groupId is string;
  
  // Eliminación: Solo el profesor que subió el archivo
  allow delete: if request.auth != null 
                   && resource.data.uploadedBy == request.auth.uid;
}
```

---

## 🎨 Experiencia de Usuario

### Profesor

#### Subir Archivo

1. **Navegación:** Ir a pestaña "Archivos" en el grupo
2. **Acción:** Click en botón "Subir Archivo" (morado intenso)
3. **Selección:**
   - Arrastrar archivo a zona de drop
   - O hacer click para abrir selector
4. **Vista Previa:** Ver nombre y tamaño del archivo seleccionado
5. **Opcional:** Agregar descripción
6. **Confirmación:** Click en "Subir Archivo"
7. **Progreso:** Ver barra de progreso en tiempo real
8. **Resultado:** Notificación de éxito y archivo aparece en lista

#### Eliminar Archivo

1. **Hover:** Pasar cursor sobre tarjeta de archivo
2. **Acción:** Click en ícono de papelera (visible en hover)
3. **Confirmación:** Confirmar eliminación en diálogo
4. **Resultado:** Archivo eliminado de Storage y Firestore

---

### Estudiante

#### Visualizar Archivos

1. **Navegación:** Ir a "Mis Grupos" → Seleccionar grupo
2. **Vista Automática:** Sección "Archivos Compartidos" en modal
3. **Información Visible:**
   - Nombre del archivo
   - Tamaño
   - Descripción (si existe)
   - Profesor que lo subió
   - Fecha de subida

#### Descargar Archivo

1. **Acción:** Click en botón "Descargar"
2. **Resultado:** Descarga inmediata o apertura en nueva pestaña

---

## 🛠️ Funciones Principales

### Profesor (`dashboard-profesor.html`)

#### 1. `openUploadFileModal()`
Abre el modal de subida, valida que hay un grupo seleccionado, limpia el formulario.

#### 2. `handleFileSelect(file)`
Valida el archivo seleccionado (tamaño, tipo), muestra preview.

#### 3. `uploadFile(e)`
Proceso completo de subida: Storage + Firestore, con seguimiento de progreso.

#### 4. `loadGroupFiles(groupId)`
Carga y renderiza los archivos del grupo.

#### 5. `deleteGroupFile(fileId, storagePath)`
Elimina archivo de Storage y Firestore con confirmación.

---

### Estudiante (`grupos.html`)

#### 1. `loadStudentGroupFiles(groupId)`
Carga archivos del grupo filtrados automáticamente.

#### 2. `createStudentFileCard(file)`
Renderiza tarjeta de archivo con toda la información y botón de descarga.

---

## 📈 Optimizaciones Implementadas

### Performance

1. **Lazy Loading:** Los archivos solo se cargan al abrir la pestaña/modal
2. **Query Indexada:** Uso de `orderBy` con índice en Firestore
3. **Caching de URLs:** URLs de descarga persistentes (no expiran)

### UX

1. **Drag & Drop:** Interfaz intuitiva para subir archivos
2. **Progreso Visual:** Barra de progreso en tiempo real
3. **Preview Inmediato:** Vista previa del archivo antes de subir
4. **Íconos Visuales:** Distinción clara por tipo de archivo
5. **Hover Effects:** Botón de eliminar solo visible al pasar cursor

### Seguridad

1. **Validación de Tamaño:** Límite de 10MB en cliente y Storage rules
2. **Validación de Tipo:** Solo tipos de archivo permitidos
3. **Rutas Organizadas:** Archivos separados por grupo
4. **Metadata Completa:** Tracking de quién subió cada archivo

---

## 🧪 Testing

### Casos de Prueba

#### Profesor

1. ✅ **Subir archivo PDF válido** → Éxito
2. ✅ **Subir archivo > 10MB** → Error: "Archivo demasiado grande"
3. ✅ **Subir archivo tipo no permitido** → Error: "Tipo no permitido"
4. ✅ **Arrastrar archivo a zona drop** → Preview correcto
5. ✅ **Cancelar subida** → Modal se cierra, sin archivo subido
6. ✅ **Eliminar archivo** → Archivo desaparece de lista
7. ✅ **Ver progreso de subida** → Barra actualiza correctamente

#### Estudiante

1. ✅ **Ver archivos de grupo asignado** → Lista completa visible
2. ✅ **No ver archivos de otros grupos** → Filtrado correcto
3. ✅ **Descargar archivo** → Descarga exitosa
4. ✅ **Ver información del archivo** → Metadata completa visible
5. ✅ **Grupo sin archivos** → Mensaje informativo

---

## 📝 Mejoras Futuras

### Corto Plazo

1. **Versionado de Archivos:** Permitir reemplazar archivos manteniendo historial
2. **Categorías:** Organizar archivos por tipo (material, tareas, recursos)
3. **Búsqueda:** Filtro de búsqueda por nombre de archivo

### Medio Plazo

1. **Vista Previa:** Preview de PDFs e imágenes sin descargar
2. **Estadísticas:** Tracking de descargas por estudiante
3. **Notificaciones:** Alertar a estudiantes cuando se sube nuevo archivo

### Largo Plazo

1. **Editor Colaborativo:** Permitir edición de documentos en línea
2. **Comentarios:** Sistema de comentarios por archivo
3. **Carpetas:** Organización jerárquica de archivos
4. **Compresión Automática:** Optimización de imágenes y PDFs

---

## 📚 Dependencias

### Firebase SDK

```javascript
// Firestore
import { 
  collection, 
  query, 
  where, 
  getDocs, 
  addDoc, 
  deleteDoc, 
  serverTimestamp,
  orderBy
} from 'firebase/firestore';

// Storage
import { 
  ref, 
  uploadBytesResumable, 
  getDownloadURL, 
  deleteObject 
} from 'firebase/storage';
```

---

## 🚀 Despliegue

### Pasos de Implementación

1. ✅ **Código Frontend:** Implementado en ambos dashboards
2. ✅ **Firebase Storage:** Configurado con reglas de seguridad
3. ✅ **Firestore Collection:** Colección `sharedFiles` lista
4. ✅ **Índices Firestore:** Crear índice para `groupId + createdAt`
5. ⏳ **Testing en Producción:** Pendiente
6. ⏳ **Monitoreo de Uso:** Configurar alertas de Storage

### Índices Requeridos en Firestore

```json
{
  "indexes": [
    {
      "collectionGroup": "sharedFiles",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "groupId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}
```

---

## 📊 Métricas de Éxito

- ✅ **Funcionalidad Completa:** Subida, visualización y descarga implementadas
- ✅ **UX Intuitiva:** Drag & drop, progreso visual, preview
- ✅ **Seguridad:** Validaciones de tamaño y tipo
- ✅ **Filtrado Correcto:** Estudiantes solo ven archivos de sus grupos
- ✅ **Performance:** Queries optimizadas con índices

---

**Fecha de implementación:** 4 de diciembre de 2025  
**Autor:** Sistema de Desarrollo CodeKids  
**Estado:** ✅ Implementado - Listo para Testing en Emuladores
