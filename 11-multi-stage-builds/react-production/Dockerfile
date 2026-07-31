# ==========================================
# STAGE 1: Build the React application
# ==========================================
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files first (caching optimization)
COPY package*.json ./

# Install all dependencies (including dev)
RUN npm install

# Copy application source
COPY . .

# Build the production bundle
RUN npm run build

# ==========================================
# STAGE 2: Serve with Nginx (minimal)
# ==========================================
FROM nginx:alpine AS production

# Copy the built React app from the build stage
COPY --from=build /app/build /usr/share/nginx/html

# Copy custom nginx config (optional)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Nginx runs on port 80 by default
EXPOSE 80

# Nginx base image already has CMD to start nginx

