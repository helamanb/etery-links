# etery-links

Sirve `link.etery.app` — los vínculos cortos que genera el panel de Etery
(Admin → Avanzado → Vínculos).

Un enlace tiene la forma `https://link.etery.app/l/{slug}` y hace tres cosas
distintas según quién lo abra:

| Quién lo abre | Qué pasa |
|---|---|
| Alguien con Etery instalada | Android abre la app directamente en el destino. **Esta web ni se carga.** |
| Alguien sin la app | Cae en `404.html` y ve el botón de descarga |
| Un navegador de escritorio | Igual: página de descarga |

## Qué hay aquí

| Fichero | Para qué |
|---|---|
| `.well-known/assetlinks.json` | Prueba de que la app puede abrir enlaces de este dominio |
| `.nojekyll` | **Imprescindible.** Sin esto GitHub Pages no publica `.well-known` |
| `404.html` | Página de recogida: sirve cualquier ruta desconocida, incluidas las `/l/{slug}` |
| `index.html` | Lo mismo, para la raíz |
| `CNAME` | El dominio: `link.etery.app` |

## Dos cosas que rompen esto en silencio

**1. Borrar `.nojekyll`.** GitHub Pages pasa los repos por Jekyll, y Jekyll
ignora todo lo que empieza por punto — incluida la carpeta `.well-known`. El
`assetlinks.json` se vería aquí en el repo pero daría 404 en la web, y entonces
Android deja de verificar el dominio: los enlaces se abrirían en el navegador
en vez de en la app. Es el fallo más común de este montaje y no avisa de nada.

**2. Que falte la huella de Play.** Google **re-firma** la app al publicarla,
así que la huella que valida en los móviles de la gente es la del certificado
de Google, no la del keystore de subida. Si solo está la de subida, los enlaces
funcionan en el emulador y fallan en producción.

Huellas que debe llevar `sha256_cert_fingerprints`:

- **Certificado de subida** (`upload-keystore.jks`, alias `upload`) — ya está.
  Sirve para probar en local y para las builds firmadas a mano.
- **Certificado de firma de la app** (lo genera Google) — Play Console →
  Configuración → Integridad de la app → Firma de apps → SHA-256 del bloque
  *Certificado de firma de la app*. **Pendiente de añadir.**

Las dos pueden convivir en el array, y deben: así funciona antes y después de
pasar por Play.

## Comprobar que está bien

El fichero tiene que responder 200 y con `content-type` de JSON:

```
curl -sI https://link.etery.app/.well-known/assetlinks.json
```

Y la verificación completa, con la herramienta de Google:

```
https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://link.etery.app&relation=delegate_permission/common.handle_all_urls
```

En el móvil, para ver si Android dio el dominio por verificado:

```
adb shell pm get-app-links com.aetherborea.eter
```

Debe decir `verified` para `link.etery.app`.

## Lo que NO hace esta web

No resuelve los slugs. No sabe a dónde apunta `verano2026` ni le hace falta:
eso lo consulta la app contra Supabase (`resolve_link`). Aquí solo hay HTML
estático, y por eso puede vivir en GitHub Pages sin coste.
