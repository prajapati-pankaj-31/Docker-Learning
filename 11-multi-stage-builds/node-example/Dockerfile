# ==========================================
# STAGE 1: Install dependencies (full SDK)
# ==========================================
FROM node:18 AS dependencies

WORKDIR /app

# Copy only package files first (leverage Docker cache)
COPY package*.json ./

# Install ALL dependencies (including dev)
RUN npm install

# ==========================================
# STAGE 2: Build / Prepare (optional step)
# ==========================================
FROM node:18 AS build

WORKDIR /app

# Copy node_modules from the dependencies stage
COPY --from=dependencies /app/node_modules ./node_modules

# Copy application source code
COPY . .

# Run any build steps (compilation, transpilation, etc.)
RUN npm run build

# ==========================================
# STAGE 3: Production runtime (minimal)
# ==========================================
FROM node:18-alpine AS production

WORKDIR /app

# Install only production dependencies
COPY package*.json ./
RUN npm install --only=production

# Copy application code from build stage
COPY --from=build /app/app.js ./app.js

EXPOSE 3000

# Use node directly (not npm) for better signal handling
CMD ["node", "app.js"]

