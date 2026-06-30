# =========================================================
# Etapa 1: Builder
# Procesa los archivos estáticos sustituyendo placeholders
# =========================================================
FROM alpine:3.19 AS builder

# Instalar envsubst (viene en gettext)
RUN apk add --no-cache gettext

WORKDIR /build

# Copiar archivos del frontend
COPY index.html app.js default.conf ./

# Recibir la URL de la API como argumento de build
ARG API_URL=http://localhost:3000/api

# Sustituir el placeholder __API_URL__ en app.js con el valor real
RUN envsubst '$API_URL' < app.js > app.processed.js && \
    mv app.processed.js app.js

# =========================================================
# Etapa 2: Runtime (Nginx)
# Sirve los archivos estáticos con Nginx
# =========================================================
FROM nginx:alpine AS runtime

# Limpiar contenido por defecto de Nginx
RUN rm -rf /usr/share/nginx/html/*

# Copiar archivos procesados desde el builder
COPY --from=builder /build/index.html /usr/share/nginx/html/
COPY --from=builder /build/app.js /usr/share/nginx/html/
COPY --from=builder /build/default.conf /etc/nginx/conf.d/default.conf

# Health check nativo
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
