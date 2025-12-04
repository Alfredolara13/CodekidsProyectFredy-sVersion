# 🎮 Sistema de Experiencia (XP) - CodeKids

## ✅ Implementación Completa

El sistema de XP ahora está **completamente funcional** y persiste correctamente en Firestore.

---

## 📊 Cómo Funciona

### 1. **Fuentes de XP**

Los estudiantes ganan XP al realizar las siguientes acciones:

| Acción | XP Ganado | Función |
|--------|-----------|---------|
| Completar una lección | **100 XP** | `window.addXP(100, 'lesson_complete')` |
| Ver un video del catálogo | **50 XP** | `window.addXP(50, 'video_complete')` |
| Entregar una tarea | **50 XP** | `window.addXP(50, 'task_submit')` |
| Completar un juego | **25 XP** | `window.addXP(25, 'game_complete')` |

### 2. **Sistema de Niveles**

El nivel se calcula automáticamente basado en el XP total acumulado:

```javascript
const LEVEL_THRESHOLDS = [
  100,   // Nivel 1 → 2
  150,   // Nivel 2 → 3
  250,   // Nivel 3 → 4
  350,   // Nivel 4 → 5
  500,   // Nivel 5 → 6
  650,   // Nivel 6 → 7
  800,   // Nivel 7 → 8
  1000,  // Nivel 8 → 9
  1200   // Nivel 9 → 10 (Nivel Máximo)
];
```

**Ejemplo:**
- Un estudiante con **400 XP** estará en **Nivel 4**
- Necesitará **350 XP más** para alcanzar el Nivel 5

### 3. **Marcos Desbloqueables**

Los marcos se desbloquean automáticamente al alcanzar ciertos niveles:

| Marco | Nivel Requerido |
|-------|----------------|
| 🥉 Marco Bronce | Nivel 2 |
| 🥈 Marco Plata | Nivel 5 |
| 🥇 Marco Oro | Nivel 8 |
| 💎 Marco Diamante | Nivel 10 |

---

## 💾 Persistencia de Datos

### **Al Ganar XP**
Cada vez que un estudiante gana XP:

1. ✅ Se actualiza el estado local (`window.userState`)
2. ✅ Se guarda en `localStorage` (copia de respaldo)
3. ✅ **Se guarda INMEDIATAMENTE en Firestore**
   ```javascript
   await updateDoc(userRef, {
     xp: newXP,
     nivel: newLevel,
     unlockedFrames: [...],
     lastXPUpdate: new Date().toISOString()
   });
   ```

### **Al Hacer Logout**
Cuando el estudiante cierra sesión:

1. ✅ Se guarda el estado en `localStorage`
2. ✅ **Se guarda el XP y nivel final en Firestore**
3. ✅ Se registra la fecha del último logout

### **Al Hacer Login**
Cuando el estudiante inicia sesión:

1. ✅ Se carga el XP desde Firestore
2. ✅ Se actualiza la barra de progreso del header
3. ✅ Se sincroniza con `localStorage`

---

## 🔧 Funciones Principales

### **Sumar XP**
```javascript
// Sumar XP con notificación
await window.addXP(100, 'lesson_complete');

// Los tipos de fuente disponibles son:
// - 'lesson_complete'
// - 'video_complete'
// - 'task_submit'
// - 'game_complete'
// - 'generic'
```

### **Completar Lección**
```javascript
// Esta función ya incluye la suma de XP automáticamente
await window.completeLesson(lessonId, 100);
```

### **Completar Video del Catálogo**
```javascript
// En dashboard-lecciones.js ya está implementado
// Se llama automáticamente al marcar como completado
await window.addXP(50, 'video_complete');
```

### **Completar Juego**
```javascript
await window.completeGame(gameId, score);
// Otorga 25 XP automáticamente
```

### **Cargar XP desde Firestore**
```javascript
// Esto se hace automáticamente al iniciar sesión
await window.loadXPFromFirestore(userId);
```

---

## 📱 Interfaz de Usuario

### **Header de Gamificación**
El header muestra:
- **Nivel actual** del estudiante
- **XP actual hacia el siguiente nivel**
- **Barra de progreso visual**

Se actualiza automáticamente cada vez que se gana XP.

### **Notificaciones Toast**
Cada vez que se gana XP, aparece una notificación en la esquina superior derecha:

```
⭐ +100 XP
¡Lección completada!
```

La notificación desaparece automáticamente después de 3 segundos.

---

## 🎯 Casos de Uso

### **Ejemplo 1: Estudiante Completa una Lección**
1. Estudiante ve el contenido de la lección
2. Hace clic en "Completar y Continuar"
3. Sistema ejecuta: `await completeLesson(lessonId, 100)`
4. Se suma 100 XP
5. Se guarda en Firestore inmediatamente
6. Aparece notificación: "+100 XP ¡Lección completada!"
7. Se actualiza el nivel si corresponde

### **Ejemplo 2: Estudiante Ve Videos del Catálogo**
1. Estudiante ve un video completo
2. Hace clic en "Marcar como completado"
3. Sistema ejecuta: `await addXP(50, 'video_complete')`
4. Se suma 50 XP
5. Se guarda en Firestore
6. Aparece notificación: "+50 XP"

### **Ejemplo 3: Estudiante Cierra Sesión**
1. Estudiante hace clic en "Cerrar Sesión"
2. Sistema guarda XP final en Firestore
3. Se registra fecha de logout
4. Se cierra la sesión
5. Al volver a iniciar sesión, su XP está intacto

---

## 🧪 Cómo Probar

### **Probar Suma de XP**
1. Inicia sesión como estudiante
2. Abre la consola del navegador (F12)
3. Ejecuta:
   ```javascript
   await window.addXP(100, 'test');
   ```
4. Verifica que:
   - Aparezca la notificación
   - Se actualice el header
   - Se guarde en Firestore (verifica en Firebase Console)

### **Probar Persistencia**
1. Suma XP con el método anterior
2. Anota el XP total y nivel actual
3. Cierra sesión
4. Vuelve a iniciar sesión
5. Verifica que el XP y nivel sean los mismos

### **Probar Completar Lección**
1. Ve a la sección "Lecciones"
2. Abre una lección
3. Haz clic en "Completar y Continuar"
4. Verifica que se sumen 100 XP

---

## 📋 Estructura en Firestore

```javascript
users/{userId} {
  xp: 450,                    // XP total acumulado
  nivel: 4,                   // Nivel actual (calculado)
  unlockedFrames: [           // Marcos desbloqueados
    'marco_bronce',
    'marco_plata'
  ],
  lastXPUpdate: '2025-12-03T...',  // Última vez que ganó XP
  lastLogout: '2025-12-03T...',    // Último logout
  
  studentProfile: {
    completedLessons: [...],   // IDs de lecciones completadas
    totalPoints: 450,          // Puntos totales (sincronizado con xp)
    badges: [...]              // Insignias ganadas
  }
}
```

---

## 🐛 Debugging

### **Ver XP Actual en Consola**
```javascript
console.log('XP:', window.userState?.state?.xp);
console.log('Nivel:', window.userState?.state?.nivel);
```

### **Ver Estado Completo**
```javascript
console.log(window.userState?.getState());
```

### **Verificar Sincronización con Firestore**
```javascript
const user = window.auth.currentUser;
const snap = await window.db.collection('users').doc(user.uid).get();
console.log('Firestore:', snap.data());
```

---

## ✨ Características Adicionales

### **Modal de Subida de Nivel**
Cuando un estudiante sube de nivel, se puede mostrar un modal especial:
```javascript
window.showLevelUpModal(newLevel);
```
*(Actualmente desactivado por defecto para no interrumpir)*

### **Verificar si un Marco está Desbloqueado**
```javascript
import { isFrameUnlocked } from './gamification.js';
const isUnlocked = isFrameUnlocked('marco_oro', nivel);
```

---

## 🚀 Próximas Mejoras

- [ ] Sistema de rachas diarias (bonus de XP)
- [ ] Multiplicadores de XP por eventos especiales
- [ ] Tabla de clasificación (leaderboard)
- [ ] Logros y medallas especiales
- [ ] Sistema de recompensas por milestones

---

## 📞 Soporte

Si tienes problemas con el sistema de XP:

1. Verifica que Firebase esté inicializado
2. Revisa la consola del navegador para errores
3. Asegúrate de que `window.addXP` esté disponible
4. Verifica los logs con:
   ```javascript
   localStorage.setItem('debug', 'true');
   ```

---

**Última actualización:** 3 de diciembre de 2025
**Versión:** 2.0 - Sistema Completo y Funcional
