# ⚽ Quiniela Mundial 2026

App de quinielas del Mundial 2026 con grupos de amigos, ranking global y bolsa acumulada.

---

## 📁 Archivos del proyecto

| Archivo | Qué hace |
|---|---|
| `index.html` | Login y registro de usuarios |
| `dashboard.html` | Pantalla principal |
| `quiniela.html` | Predicciones de partidos |
| `grupo.html` | Crear/unirse a grupos de amigos |
| `ranking.html` | Ranking global |
| `admin.html` | Panel de administrador |

---

## 🚀 Cómo subir a GitHub y publicar

### Paso 1 — Crear el repositorio en GitHub

1. Entra a [github.com](https://github.com)
2. Clic en el botón verde **New**
3. Nombre del repo: `quiniela-mundial`
4. Selecciona **Public**
5. Clic en **Create repository**

### Paso 2 — Subir los archivos

1. En la página de tu nuevo repo, clic en **uploading an existing file**
2. Arrastra todos los archivos HTML a la zona de carga
3. Escribe un mensaje: "primer commit"
4. Clic en **Commit changes**

### Paso 3 — Activar GitHub Pages

1. Ve a **Settings** en tu repo
2. Clic en **Pages** en el menú izquierdo
3. En **Source**, selecciona **Deploy from a branch**
4. En **Branch**, selecciona **main** y carpeta **/ (root)**
5. Clic en **Save**

Tu app estará disponible en:
`https://TU_USUARIO.github.io/quiniela-mundial`

(Tarda 1-2 minutos en activarse la primera vez)

---

## 🔥 Configurar Firebase

### Reglas de Firestore

En Firebase → **Firestore Database** → **Rules**, reemplaza todo con esto:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Usuarios — solo pueden leer/editar su propio perfil
    match /users/{userId} {
      allow read: if request.auth != null;
      allow create: if request.auth.uid == userId;
      allow update: if request.auth.uid == userId
        && !request.resource.data.diff(resource.data).affectedKeys().hasAny(['isAdmin']);
    }

    // Quinielas, partidos y equipos — cualquier usuario autenticado puede leer
    match /pools/{poolId} {
      allow read: if request.auth != null;
      allow write: if isAdmin();
    }

    match /matches/{matchId} {
      allow read: if request.auth != null;
      allow write: if isAdmin();
    }

    match /teams/{teamId} {
      allow read: if request.auth != null;
      allow write: if isAdmin();
    }

    // Predicciones — solo el dueño puede crear, nadie puede editar
    match /predictions/{predId} {
      allow read: if request.auth != null;
      allow create: if request.auth.uid == resource.data.userId
        || request.auth.uid == request.resource.data.userId;
      allow update, delete: if false;
    }

    // Puntos — solo el admin puede escribir
    match /matchPoints/{pointId} {
      allow read: if request.auth != null;
      allow write: if isAdmin();
    }

    // Grupos — cualquier usuario autenticado puede crear y unirse
    match /groups/{groupId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null
        && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['members']);
    }

    function isAdmin() {
      return request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isAdmin == true;
    }
  }
}
```

Clic en **Publish**.

---

## ⚙️ Primer uso — configurar el admin

1. Entra a tu app y crea tu cuenta normalmente
2. Ve a Firebase → **Firestore Database**
3. Busca la colección **users** y abre tu documento (tiene tu uid)
4. Edita el campo `isAdmin` y cámbialo de `false` a `true`
5. Guarda
6. Ahora verás el link "Panel de administrador" en tu dashboard

---

## 📋 Cómo cargar datos del Mundial

Una vez configurado como admin:

1. Ve al **Panel de administrador**
2. Tab **Equipos** → Clic en **"Cargar todos los equipos"** (carga los 48 de una vez)
3. Tab **Quinielas** → Crea la primera: "Fase de grupos"
4. Tab **Partidos** → Agrega los partidos uno por uno

---

## 🏆 Sistema de puntos

| Resultado | Puntos |
|---|---|
| Marcador exacto (ej. predijiste 2-1 y fue 2-1) | +3 pts |
| Ganador o empate correcto (ej. predijiste 2-1 y fue 3-1) | +1 pt |
| Error | 0 pts |

---

## 💡 Flujo normal de uso

1. Admin crea quiniela "Fase de grupos"
2. Usuarios se registran y hacen sus predicciones
3. Después de cada partido, admin actualiza el marcador en el panel
4. El sistema calcula puntos automáticamente
5. Rankings se actualizan en tiempo real

---

## 👥 Grupos de amigos

- Cualquier usuario puede crear un grupo y compartir el código de invitación
- El creador define el monto de entrada (solo referencia, la app no cobra)
- La bolsa se calcula sola: `integrantes × monto`
- El ranking del grupo se actualiza con cada partido terminado
