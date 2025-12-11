# backend/Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --legacy-peer-deps

# Copy only .env first for Prisma
COPY .env ./

# Copy the rest of the backend code
COPY . .

# Generate Prisma client with correct DATABASE_URL
RUN npx prisma generate

# Build the backend
RUN npm run build

EXPOSE 5000

CMD ["npm", "run", "start:prod"]

