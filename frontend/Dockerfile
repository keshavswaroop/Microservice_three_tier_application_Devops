FROM node:20-alpine AS builder 

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

FROM nginx:1.27-alpine

RUN rm -rf /usr/share/nginx/html/*

# COPY --from=builder /app/package*.json ./
COPY --from=builder /app/dist ./usr/share/nginx/html      
# as we are using vite, so the build will be stored in /app/dist

# COPY --from=builder /app/src ./src 

EXPOSE 80

ENTRYPOINT [ "nginx", "-g", "daemon off;" ]