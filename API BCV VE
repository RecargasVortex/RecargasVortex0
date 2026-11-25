// Cloudflare Worker: consulta la tasa BCV en Cotizave y se la entrega a tu página.
// La clave NO está aquí. Se guarda como "Secreto" en Cloudflare con el nombre COTIZAVE_API_KEY.
// Opcional: variable de texto ALLOWED_ORIGIN con el dominio de tu web (ej. https://vortex.pages.dev).
// Endpoint (documentación de Cotizave): GET https://api.cotizave.com/v2/fx/VE/rates/ve-bcv con cabecera X-API-Key

let cache = { at: 0, body: null };            // caché de 10 minutos para ahorrar tu cuota
const TTL_MS = 10 * 60 * 1000;

export default {
  async fetch(request, env) {
    const cors = {
      "Access-Control-Allow-Origin": env.ALLOWED_ORIGIN || "*",
      "Content-Type": "application/json",
      "Cache-Control": "public, max-age=300"
    };
    if (request.method === "OPTIONS") return new Response(null, { headers: cors });

    if (cache.body && Date.now() - cache.at < TTL_MS) return new Response(cache.body, { headers: cors });

    try {
      if (!env.COTIZAVE_API_KEY) throw new Error("falta el secreto COTIZAVE_API_KEY");
      const r = await fetch("https://api.cotizave.com/v2/fx/VE/rates/ve-bcv", {
        headers: { "X-API-Key": String(env.COTIZAVE_API_KEY).trim(), "Accept": "application/json" },
        signal: AbortSignal.timeout(10000)
      });
      if (!r.ok) throw new Error("Cotizave respondió " + r.status);
      const d = await r.json();
      if (typeof d.mid !== "number" || d.stale === true) throw new Error("dato no válido o desactualizado");

      // Solo se expone lo necesario; la clave nunca sale de aquí
      const body = JSON.stringify({ mid: d.mid, updated_at: d.updated_at, effective_date: d.effective_date || null, stale: false });
      cache = { at: Date.now(), body };
      return new Response(body, { headers: cors });
    } catch (e) {
      // Si falla, la página usa su tasa de respaldo
      return new Response(JSON.stringify({ error: "tasa_no_disponible" }), { status: 502, headers: cors });
    }
  }
};
