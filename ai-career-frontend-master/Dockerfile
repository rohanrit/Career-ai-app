# Stage 1: Build the React app
FROM node:22-alpine AS builder

WORKDIR /app

ARG VITE_SERVER_URL=http://localhost:5000
ENV VITE_SERVER_URL=$VITE_SERVER_URL

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Serve the built files with Nginx
FROM nginx:alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
