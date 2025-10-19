# La Nit de les Ànimes (Biar) – App de Inscripciones

Formulario simple con backend para gestionar inscripciones por sesión con **cupo (25)**, **lista de espera** y **correo automático**. Incluye **cancelación con promoción** desde la lista de espera y **exportación CSV** con token de admin.

---

## 🧱 Tecnologías
- **Node.js + Express** (API y servidor estático)
- **SQLite** (archivo local `event.db`)
- **Nodemailer** (envío de correo vía SMTP)
- **Frontend**: HTML + JS sin framework (accesible y ligero)

---

## 🗓️ Sesiones incluidas (seed)
- 31 de octubre de 2025, 20:30–21:30 — ID: `2025-10-31-2030`
- 2 de noviembre de 2025, 20:00–21:30 — ID: `2025-11-02-2000`
- 2 de noviembre de 2025, 21:30–23:00 — ID: `2025-11-02-2130`

Cada sesión admite **25** plazas. Al llenarse, nuevas inscripciones pasan a **lista de espera**.

---

## 📁 Estructura del proyecto
```
nit-animes-biar/
├─ .github/workflows/ci.yml        # (opcional) CI básico
├─ .gitignore                      # evita subir .env, node_modules, event.db
├─ .env.example                    # plantilla de variables (sin secretos)
├─ index.js                        # backend Express + SQLite + correo
├─ package.json                    # dependencias/scripts
└─ public/
   └─ index.html                   # formulario accesible (frontend)
```

---

## 🚀 Puesta en marcha (local)
1) Requisitos: Node.js 18+ (recomendado 20+).

2) Instala dependencias:
```bash
npm install
```

3) Crea un archivo `.env` (NO se sube a Git) copiando `.env.example` y rellenando valores reales:
```
SMTP_HOST=smtp.tu-proveedor.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=usuario_smtp
SMTP_PASS=contraseña_smtp
MAIL_FROM="La Nit de les Ànimes <no-reply@biar.es>"
PORT=3000
ADMIN_TOKEN=pon_aqui_un_token_largo_y_unico
```

> Gmail: `SMTP_HOST=smtp.gmail.com`, `SMTP_PORT=465`, `SMTP_SECURE=true` y **Contraseña de aplicación**.

4) Inicia el servidor:
```bash
node index.js
```
Abre: `http://localhost:3000`

---

## 🔌 Endpoints principales

### 1) Consultar disponibilidad
`GET /api/sessions`
- Devuelve por sesión: `remaining`, `status` (`disponible` / `lleno`), `waitlist`, etc.

### 2) Registrar inscripción
`POST /api/register`
```json
{
  "full_name": "Nombre Apellidos",
  "email": "persona@correo.com",
  "phone": "+34 600 123 123",
  "session_id": "2025-11-02-2000",
  "hp": ""  // honeypot, dejar vacío
}
```
- Si hay plaza: `status = confirmed` + **correo de confirmación**.
- Si está lleno: `status = waitlist` + **correo de lista de espera**.
- Evita duplicados por `email` en la misma sesión.

### 3) Cancelar inscripción (con promoción automática)
`POST /api/cancel`
```json
{ "email": "persona@correo.com", "phone": "+34 600 123 123", "session_id": "2025-11-02-2000" }
```
- Si la inscripción cancelada era **confirmed**, asciende al primer `waitlist` y le envía **correo de plaza confirmada**.

### 4) Exportar CSV (admin)
`GET /api/export.csv[?session_id=ID]`
- Requiere cabecera: `X-Admin-Token: <ADMIN_TOKEN>`
- Ejemplos:
```bash
# Todas las inscripciones
curl -H "X-Admin-Token: TU_TOKEN" http://localhost:3000/api/export.csv -o inscripciones.csv

# Solo una sesión
curl -H "X-Admin-Token: TU_TOKEN" "http://localhost:3000/api/export.csv?session_id=2025-10-31-2030" -o inscripciones_31oct.csv
```

---

## ✉️ Plantillas de correo
- **Confirmación de inscripción** (si hay plaza):
  > Asunto: Confirmación de inscripción – La Nit de les Ànimes (Biar)
- **Lista de espera** (si está lleno):
  > Asunto: Lista de espera – La Nit de les Ànimes (Biar)
- **Cancelación** (para quien cancela):
  > Asunto: Cancelación confirmada – La Nit de les Ànimes (Biar)
- **Promoción desde lista de espera**:
  > Asunto: ¡Plaza confirmada! – La Nit de les Ànimes (Biar)

Puedes editar los textos en `index.js` (función `sendMail` / plantillas HTML).

---

## 🧠 Cómo funciona el cupo y la lista de espera
- **Cupo por sesión**: 25. Se calcula `remaining = capacity - confirmed`.
- **Registro**:
  - Si `remaining > 0` → `status = confirmed`.
  - Si `remaining == 0` → `status = waitlist` con `position` incremental.
- **Cancelación**:
  - Se elimina la inscripción.
  - Si era `confirmed`, el **primer** en `waitlist` pasa a `confirmed` y recibe correo.

---

## 🛡️ Privacidad y seguridad básicas
- Solo se piden **datos mínimos**: nombre, correo y teléfono.
- Campo **honeypot** para bots.
- `UNIQUE(session_id, email)` evita duplicados.
- El endpoint de CSV requiere **token** en cabecera.

> Para producción con más tráfico, usa **PostgreSQL gestionado** en lugar de SQLite.

---

## 🌐 Despliegue recomendado

### Opción A: Render.com
1. Conecta el repo de GitHub y crea un **Web Service**.
2. **Build Command**: `npm install`
3. **Start Command**: `node index.js`
4. Añade **Environment Variables** (copiando `.env.example`).
5. **Disco persistente**: añade un Disk y móntalo en la ruta del proyecto para no perder `event.db`.

### Opción B: Railway.app
1. Deploy desde GitHub.
2. Variables en **Project → Variables**.
3. **Start Command**: `node index.js`.
4. Añade un **Volume** para persistir `event.db`.

> **GitHub Pages** no es válido (requiere backend).

---

## ✅ Checklist de publicación
- [ ] Repo con `.env.example` y `.gitignore` correctos.
- [ ] Variables de entorno configuradas en el proveedor.
- [ ] `node index.js` como comando de arranque.
- [ ] Volumen/disco para `event.db` configurado.
- [ ] Prueba real: inscripción confirmada, lista de espera, cancelación y promoción.
- [ ] Verifica recepción de correos.

---

## 🧩 Scripts útiles en `package.json`
```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "NODE_ENV=development node index.js"
  }
}
```

---

## 🔧 Resolución de problemas
- **No llegan correos**: revisa SMTP (host/puerto/usuario/contraseña, `SMTP_SECURE`), spam, dominio con SPF/DKIM.
- **DB se vacía tras reinicio**: faltó configurar volumen/disco persistente.
- **Error UNIQUE** al registrar: ese email ya está inscrito en esa sesión.
- **CSV devuelve 401**: confirma cabecera `X-Admin-Token` y valor del `ADMIN_TOKEN`.

---

## 📜 Licencia
Uso interno para la organización del evento. Adáptala si publicas el repositorio.
