FROM node:22 AS build

WORKDIR /app

COPY ./SAES6_ENT_PRIVATEFRONT/package.json ./SAES6_ENT_PRIVATEFRONT/package-lock.json .
RUN npm ci

COPY ./SAES6_ENT_PRIVATEFRONT .

RUN npm run build -- --configuration production


FROM nginx:alpine AS serve

COPY ./nginx.conf /etc/nginx/conf.d/default.conf

COPY --from=build /app/dist/browser /usr/share/nginx/html/

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]