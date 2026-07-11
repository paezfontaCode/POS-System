@"

\# Saman Digital - POS \& Taller



Sistema ERP/POS para gestión de telefonía móvil, accesorios y servicio técnico.



\## Documentación



\- \[Plan Maestro](./docs/PLAN\_MAESTRO.md)

\- \[Contexto Maestro](./docs/CONTEXTO\_MAESTRO.md)



\## Stack



\*\*Frontend:\*\* React 18 + Vite + TypeScript + Tailwind + shadcn/ui  

\*\*Backend:\*\* Node.js + Express + TypeScript + Prisma + Zod  

\*\*Base de Datos:\*\* PostgreSQL 16 (Docker)



\## Instalación



\\`\\`\\`bash

\# 1. Levantar base de datos

docker compose up -d



\# 2. Backend

cd backend \&\& npm install \&\& npm run dev



\# 3. Frontend (en otra terminal)

cd frontend \&\& npm install \&\& npm run dev

\\`\\`\\`



\## Estado del Proyecto



\- \[x] Planificación completa

\- \[ ] Fase 1: Cimientos (DB + Backend)

\- \[ ] Fase 2: Frontend + Inventario

\- \[ ] Fase 3: POS + Taller

\- \[ ] Fase 4: Estabilización

\- \[ ] Fase 5: Finanzas y Caja

"@ | Out-File -FilePath README.md -Encoding utf8

