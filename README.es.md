[English](README.md) | [한국어](README.ko.md) | [中文](README.zh.md) | [日本語](README.ja.md) | Español

# nopal

<p align="center">
  <img src="assets/nopal-hero-01.png" alt="nopal" width="320">
</p>

> **Orquestación de Google Workspace, impulsada por lenguaje natural.**

Convierte una frase corriente en una acción coordinada entre Gmail, Calendar, Drive, Docs, Sheets, Slides, Meet, Tasks y Chat — sin salir de Claude Code.

[Inicio rápido](#inicio-rápido) • [¿Por qué nopal?](#por-qué-nopal) • [Cómo funciona](#cómo-funciona) • [Servicios](#servicios) • [Requisitos](#requisitos)

---

## Inicio rápido

### 1. Añade el marketplace (una sola vez)

```
/plugin marketplace add https://github.com/fivetaku/gptaku_plugins.git
```

### 2. Instala nopal

```
/plugin install nopal
```

Reinicia Claude Code después de la instalación.

### 3. Configura la gws CLI (una sola vez)

nopal usa la [gws CLI](https://github.com/googleworkspace/cli) para comunicarse con Google Workspace. Instálala primero:

```bash
npm install -g @googleworkspace/cli
```

Luego ejecuta la configuración OAuth (una sola vez) en tu terminal:

```bash
gws auth setup
```

Te guía por la creación de un proyecto de GCP, la activación de las 9 APIs de Workspace y la autorización de tu cuenta de Google. Tras la configuración, inicia sesión:

```bash
gws auth login
```

Después de iniciar sesión, exporta las credenciales para que Claude Code pueda usar gws en modo headless:

```bash
gws auth export --unmasked 2>/dev/null | grep -v '^Using keyring' > ~/.config/gws/credentials.json
chmod 600 ~/.config/gws/credentials.json   # tokens en texto plano — mantenlos privados, nunca los subas al repo
```

### 4. Ejecuta

```
/nopal
```

No hacen falta argumentos: nopal te pregunta qué quieres hacer. O ve directo al grano:

```
/nopal programa un standup del equipo mañana a las 10 y envía la agenda por correo a los asistentes
/nopal revisa mis correos sin leer y resume los importantes
/nopal crea un documento de actas y compártelo con los asistentes de la semana pasada
/nopal saca los datos de ventas del Q1 de Sheets y envía un resumen al chat del equipo
```

---

## ¿Por qué nopal?

- **Un comando, cualquier servicio** — describe lo que quieres en lenguaje corriente; nopal decide qué servicios invocar y en qué orden
- **Composición dinámica** — no es una biblioteca de flujos fijos; los servicios se seleccionan y encadenan según cada petición
- **Guiado por entrevista** — si falta información, nopal pregunta antes de actuar (no después)
- **Distinción entre lectura y escritura** — las consultas de solo lectura se ejecutan al momento; las acciones de escritura y modificación siempre piden tu confirmación primero
- **Vive en Claude Code** — sin apps nuevas, sin pestañas del navegador, sin cambios de contexto
- **Las credenciales se quedan en gws** — la gws CLI gestiona el flujo OAuth. Para el uso headless exporta los tokens a un `~/.config/gws/credentials.json` local (mantenlo con `chmod 600` y nunca lo subas al repo); nopal jamás incrusta credenciales en Claude

---

## Cómo funciona

```
Tú: "programa una reunión de equipo mañana a las 14:00 y avisa por correo a los asistentes"
     │
     ▼
/nopal
     │
     ├─ ¿gws no está instalado? → intento de instalación automática / guía de configuración
     │
     └─ gws listo → comienza la orquestación
          │
          ├─ 1. Analizar la intención — ¿qué servicios hacen falta?
          ├─ 2. Entrevistar           — obtener datos en vivo y preguntar solo lo que falta
          ├─ 3. Planificar            — confirmar las acciones de escritura, omitir las de solo lectura
          ├─ 4. Ejecutar              — lanzar los comandos gws en secuencia
          └─ 5. Informar              — resumir resultados + sugerir próximos pasos
```

Las peticiones multiservicio se resuelven con naturalidad:

- "añade asistentes a la reunión de mañana y envíales el documento" → Calendar + Drive + Gmail
- "crea una newsletter con los datos de Sheets y envíala" → Sheets + Gmail
- "redacta las actas y publícalas en el espacio de Chat del equipo" → Docs + Chat

---

## Servicios

| Servicio | Qué puede hacer nopal | Comandos auxiliares |
|---------|-------------------|-----------------|
| Gmail | Enviar, leer, clasificar, vigilar | `+send`, `+triage`, `+watch` |
| Calendar | Crear eventos, consultar la agenda | `+insert`, `+agenda` |
| Drive | Subir archivos, gestionar el uso compartido | `+upload` |
| Sheets | Leer/añadir datos de hojas de cálculo | `+read`, `+append` |
| Docs | Leer y escribir documentos | `+write` |
| Slides | Crear y editar presentaciones | — |
| Chat | Enviar mensajes a espacios | `+send` |
| Tasks | Gestionar listas de tareas | — |
| Meet | Crear enlaces de reunión, obtener participantes y transcripciones | — |

---

## Problemas conocidos

| Problema | Estado | Solución |
|-------|--------|------------|
| Error 411 en Gmail trash | Corregido en gws 0.6.1+ | Usa la versión más reciente |
| Codificación del coreano en `+send` | Bug de la gws CLI | Se aplica automáticamente la codificación por API raw |
| Logs mezclados en `gws auth export` | `Using keyring backend` se cuela en el JSON | Se aplica el filtro `2>/dev/null \| grep -v '^Using keyring'` |

---

## Requisitos

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [gws CLI](https://github.com/googleworkspace/cli) — `npm install -g @googleworkspace/cli`
- Cuenta de Google Workspace + configuración OAuth (`gws auth setup` + `gws auth login`)
- Node.js 18+

> La primera vez que ejecutas `/nopal`, comprueba si gws está disponible y te guía por la configuración automáticamente.

---

## Licencia

MIT

---

<div align="center">

**No Opal needed. — No hace falta Opal.**

</div>
