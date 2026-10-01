# Imagen del CronJob frigate-restart: Alpine con el cliente SSH ya instalado.
#
# Existe porque el CronJob hacía `apk add openssh-client` en cada pasada, seis
# veces al día, y el 30/09/2026 a las 16:00Z la descarga desde dl-cdn se quedó
# colgada a 918 KB hasta que la mató el activeDeadlineSeconds: Frigate se quedó
# sin reinicio y saltó JobFailed. Con el cliente dentro de la imagen, la pasada
# ya no depende de internet, solo de la VM de Frigate.
#
# La construye, escanea y publica .github/workflows/build.yml; no se
# construye a mano.

# Base fijada por digest (índice multi-arch de alpine:3.24.2), no solo por tag:
# un tag se puede volver a empujar, un digest no.
FROM alpine:3.24.2@sha256:294b683cb724975bec92580e1e685676bd4b50bda910ddb8c51d4cabeaec77e6

# - apk upgrade: el digest congela la base, así que sin esto una reconstrucción
#   no recogería los parches de seguridad publicados después.
# - openssh-client-default es lo mínimo que trae `ssh` (arrastra scp, sftp y
#   ssh-keygen como dependencias; el metapaquete openssh-client añade más).
# - Usuario sin privilegios con entrada en /etc/passwd: ssh aborta con
#   "No user exists for uid" si el uid no existe. Home en /tmp, que en el pod
#   es un emptyDir: el rootfs va de solo lectura.
RUN apk upgrade --no-cache \
 && apk add --no-cache openssh-client-default \
 && addgroup -g 10001 restarter \
 && adduser -D -H -h /tmp -s /sbin/nologin -u 10001 -G restarter restarter

USER 10001:10001
