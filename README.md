# 🃏 NexoCards

**Marketplace peruano de cartas y coleccionables**

![Next.js](https://img.shields.io/badge/Next.js-16-black) ![TypeScript](https://img.shields.io/badge/TypeScript-Type--Safe-3178C6) ![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E) ![Tailwind](https://img.shields.io/badge/Tailwind-CSS-38B2AC) ![Vercel](https://img.shields.io/badge/Vercel-Deployed-black)

**🌐 Producto en vivo:** [nexocards.pe](https://nexocards.pe/)

> Este repositorio es una vitrina del proyecto. El código fuente es privado — NexoCards es un producto en producción con usuarios y transacciones reales, así que el repositorio de desarrollo no es público. Aquí documento el problema, las decisiones técnicas y el resultado.

## 📸 Capturas

| | | |
|---|---|---|
| ![Home](docs/screenshots/01-home.png) | ![Explorar](docs/screenshots/02-explorar.png) | ![Detalle de carta](docs/screenshots/03-detalle.png) |
| Home | Explorar con filtros | Detalle de publicación |

## 💡 El problema

En Perú no existía una plataforma centralizada para comprar, vender e intercambiar cartas y figuritas coleccionables. Los coleccionistas dependían de grupos de Facebook y WhatsApp sin una forma clara de comparar referencias de precio, revisar la reputación de vendedores o gestionar transacciones.

## ✅ La solución

NexoCards conecta a coleccionistas peruanos de Pokémon, Magic: The Gathering, Yu-Gi-Oh!, One Piece y coleccionables deportivos. Integra referencias de precio, subastas, ofertas con contraofertas, mensajería y reputación de vendedores.

## ✨ Funcionalidades destacadas

- **Búsqueda inteligente**: autocompletado por nombre, set, número, jugador o selección, manteniendo siempre la opción de publicar manualmente.
- **Coleccionables deportivos**: sección unificada para figuritas del Mundial y tarjetas deportivas, con Fútbol como categoría principal y otros deportes agrupados para mantener una experiencia relevante en Perú.
- **Conversión de moneda automática**: precio USD de TCGPlayer convertido a soles con multiplicadores configurables por el vendedor.
- **Subastas y ofertas**: pujas con incrementos fijos, precio de reserva, protección anti-sniping, y ofertas en efectivo, cartas o mixtas con contraofertas.
- **Mensajería en tiempo real**: chat directo entre compradores y vendedores con imágenes, reacciones y notificaciones.
- **Reputación**: sistema de reseñas y confiabilidad de pago que se refleja en el perfil público del vendedor.
- **Panel de administración**: gestión de usuarios, publicaciones, transacciones y cola de verificación.

## 🛠️ Stack técnico

**Frontend:** Next.js 16 (App Router, Server Components), TypeScript, Tailwind CSS, shadcn/ui, Zustand
**Backend:** Next.js API Routes, Supabase (PostgreSQL, Auth, Storage) con Row Level Security
**Integraciones:** eBay/TCGPlayer API para precios de mercado, Resend para notificaciones por correo, Google Analytics 4
**Infraestructura:** Desplegado en Vercel con CI/CD automático, dominio propio (nexocards.pe)

## 🆕 Avances de agosto de 2026

- NexoCards amplió el marketplace más allá de los TCG con una sección de coleccionables deportivos y una entrada única para Figuritas del Mundial.
- El homepage ahora rota diariamente sus publicaciones destacadas, evita duplicarlas en la sección reciente y utiliza un encuadre más cercano de las fotos.
- Se refinó la jerarquía visual de las métricas principales y se restauró el acceso contextual al panel de administración para cuentas autorizadas.
- Las referencias de eBay distinguen claramente entre anuncios activos y ventas cerradas: se muestran como orientación, con rango y tamaño de muestra, y nunca se aplican sin decisión del vendedor.

## 🏗️ Decisiones de arquitectura (a nivel general)

- **Server Components + API Routes**: se usa el App Router de Next.js para renderizado eficiente en servidor, con rutas de API dedicadas para operaciones que requieren lógica de negocio (transacciones, ofertas, subastas).
- **Row Level Security en Supabase**: en vez de validar permisos solo en la capa de aplicación, las reglas de acceso a datos viven también en la base de datos — así ningún endpoint puede accidentalmente exponer datos de otro usuario.
- **Rate limiting por IP**: en endpoints sensibles (auth, mensajería, subastas) para mitigar abuso.
- **Referencias desacopladas de terceros**: el producto combina datos actualizados periódicamente con consultas puntuales y caché, evitando depender de una sola API durante la publicación.

## 🐛 Bugs reales que encontré (y cómo los resolví)

- **Recursión infinita en políticas RLS de `listings`/`auctions`/`bids`.** La política de `listings` necesitaba leer `bids` (vía join con `auctions`) para decidir si un postor podía seguir viendo una subasta cerrada; la política de `bids`, a su vez, necesitaba leer `listings` de vuelta para decidir si el vendedor podía ver las pujas de su propia subasta. Cada lectura disparaba la evaluación de política de la otra tabla, causando errores 500 reales en producción. Tomó varias rondas de migraciones encontrar el patrón de fondo — no hay forma de que dos tablas se restrinjan mutuamente sin crear el ciclo. La solución final rompió el ciclo a propósito, relajando `bids` a lectura pública en vez de seguir intentando "arreglar" ambos lados a la vez.
- **El botón de aprobar verificación de vendedor no hacía nada.** Aprobar una solicitud en el panel de admin actualizaba solo el estado de la solicitud — nunca el perfil del vendedor —, así que el contador de "vendedores verificados" del home se quedaba en 0 sin importar cuántas solicitudes se aprobaran. Encontrado revisando el flujo completo, no solo el botón. Ya corregido, además diseñé la lógica que faltaba: verificación manual instantánea (para tiendas sin historial de ventas en la plataforma) y verificación automática por historial (ventas + rating sostenido) que solo puede *otorgar* el estado, nunca revocarlo — así una tienda verificada a mano nunca corre riesgo de perder el badge por un trigger de fondo.
- **Un precio en dólares que parecía una conversión, pero no lo era.** El detalle de una publicación mostraba un monto en USD junto al precio en soles con toda la apariencia de ser la conversión de ese precio — pero en realidad era el precio de referencia de TCGPlayer/eBay, un número completamente distinto que solo coincidía en formato. Confirmé el error contra el tipo de cambio real del día (una carta de S/. 60 mostraba ≈$21.71 cuando la conversión real era ≈$17.80) y lo reemplacé por una conversión real, cacheada, contra una API de tipo de cambio en vivo.

## 📊 Escala actual

- Marketplace en producción con transacciones reales desde 2025.
- Soporta Pokémon, Magic: The Gathering, Yu-Gi-Oh!, One Piece y coleccionables deportivos.

## 🛣️ Roadmap

Integración de pagos locales (Yape/Plin, tarjetas), sistema de escrow para subastas y ofertas, expansión a otros países de Latinoamérica.

## 👨‍💻 Mi rol

Desarrollo full-stack end-to-end: diseño de producto, arquitectura de base de datos, integración de APIs externas de pricing, sistema de subastas/ofertas, y despliegue en producción.

---

**¿Quieres ver el código?** Escríbeme a [contacto@nexocards.pe](mailto:contacto@nexocards.pe) o conéctate conmigo — puedo dar acceso puntual al repositorio privado para procesos de entrevista.
