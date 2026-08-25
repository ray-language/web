# `web` — el framework web de aplicación (M93)

> **Espejo de solo lectura** — publicado desde
> [`raylang/packages/web`](https://github.com/roberto-ayala/raylang/tree/main/packages/web);
> el desarrollo y los PRs van al monorepo.
>
> **Instalación** — en tu `ray.toml`:
>
> ```toml
> [registry]
> index = "git+https://github.com/ray-language/ray-index@main"
> ```
>
> y `ray add web` — o la dependencia directa:
> `web = "git+https://github.com/ray-language/web@v0.1.0"`.


Framework estilo **Express** escrito en raylang puro sobre `net/webserver` (el servidor HTTP de
producción, M56). Promovido desde `examples/web/framework.ray`; corre en la **VM y en el binario
nativo** (el servidor cede fibras; el intérprete no las tiene). Guía completa:
[`docs/web-framework.md`](../../docs/web-framework.md).

```rust
from web/framework import new_app, GET, listen, static_files, log_requests, text, Ctx, Res;

fn main() -> int {
    var app = new_app();
    app.log_requests();                       // JSON por petición (net/log)
    app.static_files("/assets/", "static");   // estáticos con ETag/304 (M56.9)
    app.GET("/hola/:nombre", fn(c: Ctx, r: Res) {
        r.text("hola, " + c.param("nombre"));
    });
    match (app.listen("127.0.0.1", 8080)) {
        Result.Ok(_) => 0,
        Result.Err(e) => { eprint(e); 1 },
    }
}
```

- **Enrutado** por método + patrón con parámetros (`/users/:id` → `c.param("id")`); `GET`/`POST`/
  `PUT`/`PATCH`/`DELETE` + `route(app, método, …)` genérico; HEAD enruta como GET (el servidor
  quita el cuerpo, RFC 9110).
- **Middleware** (`use_mw`): corren antes del enrutado, en orden; devolver `false` corta la cadena.
- **Respuesta encadenable**: `r.status(418).header("X-K", "v").text(…)`; `json`/`html`/`redirect`/
  `cookie`; **404 personalizable** (`not_found`).
- **Estáticos** (`static_files(prefix, dir)`): sobre `static_mount` de M56.9 — mime por extensión,
  saneo de traversal, `ETag` fuerte y `304` de revalidación. Se comprueban antes que las rutas,
  solo GET/HEAD.
- **Logging** (`log_requests`): una línea JSON por petición (`net/log`) con método, ruta, status y
  duración en ms.
- **Despliegue**: `listen` (keep-alive + límites por defecto + panic-del-handler→500, herencia de
  `webserver.serve`), `listen_tls(cert, key)` (HTTPS, M56.3), `listen_graceful(drain_ms)` (apagado
  ordenado con SIGTERM/SIGINT, M88.1b) y `listen_limits(webserver.Limits)`.

## Instalación

En tu `ray.toml` (por ruta en el monorepo; git desde el espejo publicado):

```toml
[dependencies]
web = "git+https://github.com/ray-language/web@v0.1.0"
net = "git+https://github.com/ray-language/net@v0.1.0"   # web se apoya en net/webserver y net/log
```

Demo completo: [`examples/web/framework/`](../../examples/web/framework/).
