FROM node:24-bookworm-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration production

FROM nginx:1.27-alpine
RUN apk add --no-cache gettext
COPY nginx.conf.template /etc/nginx/default.conf.template
COPY --from=build /app/dist/epoc-frontend/browser /usr/share/nginx/html
EXPOSE 10000
CMD ["/bin/sh", "-c", "envsubst '${PAYOPS_API_HOSTPORT}' < /etc/nginx/default.conf.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"]
