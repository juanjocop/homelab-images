# homelab-images

Imágenes pequeñas del homelab que no merecen un repo propio. Cada directorio
`images/<nombre>/` con un `Dockerfile` se publica como
`ghcr.io/juanjocop/<nombre>`; el workflow lo descubre solo, no hay que darlo de
alta en ningún sitio.

| Imagen | Para qué | Quién la usa |
| --- | --- | --- |
| `frigate-restart` | Alpine con `ssh`, para reiniciar Frigate por SSH | CronJob `default/frigate-restart` en `k3s-local-apps-manifests` |

## Reglas

- **Aquí no entra nada secreto.** El repo y las imágenes son públicos: claves,
  tokens y configuración privada van en el consumidor (SealedSecrets en el
  clúster), nunca en la imagen.
- **Base fijada por digest** (`FROM imagen:tag@sha256:...`), y `apk upgrade`
  en el build para que reconstruir recoja parches aunque la base esté congelada.
- **Sin root**: `USER` numérico al final, con su entrada en `/etc/passwd` si la
  herramienta la necesita (ssh, por ejemplo, aborta sin ella).
- **Tags inmutables** `YYYYMMDD-<sha7>`, nunca `latest`. El consumidor fija
  además el digest: `ghcr.io/juanjocop/<nombre>:<tag>@sha256:...`.
- Las actions del workflow van **fijadas por SHA de commit**. Para subir una,
  resolver el tag a commit (`gh api repos/<owner>/<repo>/git/ref/tags/<tag>`,
  desreferenciando si es un tag anotado) y cambiar SHA y comentario a la vez.

## Plataformas

Por defecto solo `linux/arm64`, que es lo que son los workers del clúster. Si
una imagen necesita otras, va un fichero `images/<nombre>/platforms` con la
lista (`linux/arm64,linux/amd64`). Trivy escanea la primera de la lista.

## Cómo publica

- **PR**: construye y escanea las imágenes tocadas. No publica.
- **Push a `main`**: igual, y si Trivy no encuentra HIGH/CRITICAL con parche,
  publica. Si se toca el workflow, se revalidan todas.
- **Manual** (`gh workflow run build -f image=<nombre>`): reconstruye y publica.
  Es la manera de recoger un parche de seguridad sin cambiar el Dockerfile.
- **Lunes 06:17 UTC**: reconstruye y escanea todas, sin publicar. Si sale en
  rojo, GitHub avisa por correo: lanzar la manual de esa imagen y cambiar el
  digest en el consumidor.

El digest que hay que copiar sale en el resumen del run.

Ojo: en un repo público GitHub **desactiva los workflows programados tras 60
días sin actividad** (avisa por correo antes). Si llega ese correo, reactivarlo
desde la pestaña Actions; si no, el escaneo semanal deja de existir en silencio.
