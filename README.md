# FinanceOS (FOS) - Personal Finance Management System

## Qué es
FinanceOS es una plataforma web para la gestión financiera personal que centraliza liquidez, consumos con tarjeta de crédito, cuotas, préstamos e inversiones.
Integra un simulador de metas de ahorro y un copiloto financiero con Inteligencia Artificial que analiza tus hábitos de gasto para optimizar la toma de decisiones.

## Arquitectura
```text
[ Usuario / Navegador ]
       │
       ▼ (HTTPS / SSL via Cloudflare)
[ Proxy / DNS: Cloudflare ]
       │
       ▼
[ Servidor VPS Hetzner ] ──► [ Orquestador: Coolify (Docker / Nixpacks) ]
       │
       ├───────────────────────────────┐
       ▼                               ▼
[ Frontend: React + Vite ]     [ Backend: Node.js + Express API ]
                                       │
                                       ├──────────────► [ IA: Google Gemini API ]
                                       │
                                       ▼
                       [ DB: PostgreSQL / Supabase ]
                       Host: <aqui_host_db>
                       IP: <aqui_tu_ip>
```

## Stack
* **Frontend**: React, Vite, TailwindCSS, Zustand, Lucide React.
* **Backend**: Node.js, Express API, JWT Authentication, Express-Rate-Limit, CORS.
* **Base de Datos**: PostgreSQL (alojado en Supabase) con Triggers y funciones de cálculo.
* **Infraestructura & DevOps**: Hetzner VPS, Coolify, Docker (Nixpacks), Cloudflare (DNS/SSL/Proxy).
* **Inteligencia Artificial**: Google Gemini API (`gemini-2.5-flash`).

## Incidentes / decisiones

* **Incidente 1: Descuadre contable y cuotas ausentes en resúmenes**
  * **Síntoma**: Los resúmenes de tarjetas omitían cuotas pendientes, generando un descuadre financiero.
  * **Causa**: Dispersión de lógica en el frontend intentando calcular desfases de fechas (`N-1`/`N+1`) y falta de proyección de datos en las consultas.
  * **Solución**: Adopción de la arquitectura *Single Source of Truth* (KISS). Se centralizó el cálculo de vencimientos (`due_date`) en PostgreSQL y se simplificó el frontend para que solo renderice fechas planas.

* **Incidente 2: Bloqueo masivo de peticiones legítimas (HTTP 429 - Rate Limit)**
  * **Síntoma**: Mensaje de error "Demasiadas solicitudes" al realizar acciones consecutivas.
  * **Causa**: Configuración de Rate Limit en Express que evaluaba la IP del proxy intermedio (Cloudflare) en lugar de la IP real del usuario.
  * **Solución**: Habilitación de `app.set('trust proxy', true)` e implementación de un `keyGenerator` dinámico evaluando `CF-Connecting-IP` o el token de sesión JWT.

* **Incidente 3: Fallo de despliegue automático y compilación en Nixpacks**
  * **Síntoma**: Despliegues automáticos detenidos tras pushes a GitHub o errores de compilación de binarios.
  * **Causa**: Webhooks desvinculados en GitHub y conflictos de dependencias nativas entre entornos de desarrollo y el servidor Linux x64.
  * **Solución**: Reconfiguración manual de Webhooks con Secret Token, definición explícita de dependencias nativas Linux (`@rolldown/binding-linux-x64-gnu`) y fijación de Node.js (`22.12.0`) vía `NIXPACKS_NODE_VERSION`.

## Estado real

### En Producción
* Gestión de cuentas de liquidez e inversiones integradas.
* Registro de movimientos históricos y transacciones programadas ("A Vencer").
* Resumen consolidado de tarjetas de crédito y cuotas con fechas efectivas planas.
* Módulo de préstamos con cálculo según Sistema Francés y cancelación anticipada.
* Módulo de ahorros con simulador de interés compuesto (FCI) y seguimiento de metas.
* Copiloto Financiero con IA integrado en el Dashboard principal.
* Panel de administración con control de acceso y permisos granulares por usuario.
* Despliegue automatizado vía Webhooks en Coolify con HTTPS activo.

### En Plan (Backlog)
* Automatización de backups periódicos en la nube (Google Drive).
* Servicio de notificaciones transaccionales vía Email (SMTP).
* Módulo de exportación de estados financieros y reportes en PDF y Excel.
