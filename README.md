# 🃏 NexoCards

**El marketplace líder de trading cards en Perú**

![Next.js](https://img.shields.io/badge/Next.js-16-black) ![TypeScript](https://img.shields.io/badge/TypeScript-Type--Safe-3178C6) ![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E) ![Tailwind](https://img.shields.io/badge/Tailwind-CSS-38B2AC) ![Vercel](https://img.shields.io/badge/Vercel-Deployed-black)

**🌐 Producto en vivo:** [nexocards.pe](https://nexocards.pe/)

> Este repositorio es una vitrina del proyecto. El código fuente es privado — NexoCards es un producto en producción con usuarios y transacciones reales, así que el repositorio de desarrollo no es público. Aquí documento el problema, las decisiones técnicas y el resultado.

## 📸 Capturas

| | | |
|---|---|---|
| ![Home](docs/screenshots/01-home.png) | ![Explorar](docs/screenshots/02-explorar.png) | ![Detalle de carta](docs/screenshots/03-detalle.png) |
| Home | Explorar con filtros | Detalle de publicación |

## 💡 El problema

En Perú no existía una plataforma centralizada para comprar, vender e intercambiar cartas coleccionables (Pokémon, Magic: The Gathering, Yu-Gi-Oh!, One Piece). Los coleccionistas dependían de grupos de Facebook y WhatsApp sin forma confiable de validar precios de mercado, reputación de vendedores o gestionar transacciones.

## ✅ La solución

NexoCards es un marketplace completo que conecta a la comunidad de TCG en Perú, con precios de mercado en tiempo real (vía TCGPlayer/eBay API) convertidos automáticamente a soles, sistema de subastas, ofertas con contraofertas, mensajería integrada y reputación de vendedores.

## ✨ Funcionalidades destacadas

- **Búsqueda inteligente**: autocompletado sobre una base de 24,500+ cartas indexadas entre Pokémon, Magic, Yu-Gi-Oh! y One Piece, con precios de mercado en vivo.
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

## 🏗️ Decisiones de arquitectura (a nivel general)

- **Server Components + API Routes**: se usa el App Router de Next.js para renderizado eficiente en servidor, con rutas de API dedicadas para operaciones que requieren lógica de negocio (transacciones, ofertas, subastas).
- **Row Level Security en Supabase**: en vez de validar permisos solo en la capa de aplicación, las reglas de acceso a datos viven también en la base de datos — así ningún endpoint puede accidentalmente exponer datos de otro usuario.
- **Rate limiting por IP**: en endpoints sensibles (auth, mensajería, subastas) para mitigar abuso.
- **Precios en tiempo real con fallback**: un scraper propio actualiza diariamente un caché local de 24,500+ cartas desde TCGPlayer/eBay, para no depender 100% de la disponibilidad de terceros en cada consulta.

## 🐛 Bugs reales que encontré (y cómo los resolví)

- **Recursión infinita en políticas RLS de `listings`/`auctions`/`bids`.** La política de `listings` necesitaba leer `bids` (vía join con `auctions`) para decidir si un postor podía seguir viendo una subasta cerrada; la política de `bids`, a su vez, necesitaba leer `listings` de vuelta para decidir si el vendedor podía ver las pujas de su propia subasta. Cada lectura disparaba la evaluación de política de la otra tabla, causando errores 500 reales en producción. Tomó varias rondas de migraciones encontrar el patrón de fondo — no hay forma de que dos tablas se restrinjan mutuamente sin crear el ciclo. La solución final rompió el ciclo a propósito, relajando `bids` a lectura pública en vez de seguir intentando "arreglar" ambos lados a la vez.
- **El botón de aprobar verificación de vendedor no hacía nada.** Aprobar una solicitud en el panel de admin actualizaba solo el estado de la solicitud — nunca el perfil del vendedor —, así que el contador de "vendedores verificados" del home se quedaba en 0 sin importar cuántas solicitudes se aprobaran. Encontrado revisando el flujo completo, no solo el botón. Ya corregido, además diseñé la lógica que faltaba: verificación manual instantánea (para tiendas sin historial de ventas en la plataforma) y verificación automática por historial (ventas + rating sostenido) que solo puede *otorgar* el estado, nunca revocarlo — así una tienda verificada a mano nunca corre riesgo de perder el badge por un trigger de fondo.
- **Un precio en dólares que parecía una conversión, pero no lo era.** El detalle de una publicación mostraba un monto en USD junto al precio en soles con toda la apariencia de ser la conversión de ese precio — pero en realidad era el precio de referencia de TCGPlayer/eBay, un número completamente distinto que solo coincidía en formato. Confirmé el error contra el tipo de cambio real del día (una carta de S/. 60 mostraba ≈$21.71 cuando la conversión real era ≈$17.80) y lo reemplacé por una conversión real, cacheada, contra una API de tipo de cambio en vivo.

## 📊 Escala actual

- Marketplace en producción con transacciones reales desde 2025.
- Soporta 4 juegos de cartas coleccionables (Pokémon, Magic, Yu-Gi-Oh!, One Piece).
- Base de datos de precios con 24,500+ cartas indexadas, actualizada diariamente.

## 🛣️ Roadmap

Integración de pagos locales (Yape/Plin, tarjetas), sistema de escrow para subastas y ofertas, expansión a otros países de Latinoamérica.

## 👨‍💻 Mi rol

Desarrollo full-stack end-to-end: diseño de producto, arquitectura de base de datos, integración de APIs externas de pricing, sistema de subastas/ofertas, y despliegue en producción.

---

**¿Quieres ver el código?** Escríbeme a [contacto@nexocards.pe](mailto:contacto@nexocards.pe) o conéctate conmigo — puedo dar acceso puntual al repositorio privado para procesos de entrevista.
