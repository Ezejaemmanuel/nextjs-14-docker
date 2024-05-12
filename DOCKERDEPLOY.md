### GitHub Markdown Version: Advanced Docker Setup for Next.js 14

Below is the GitHub markdown version of the tutorial on setting up Docker for a Next.js 14 application using advanced techniques. This markdown is formatted for clarity and emphasis, suitable for a README file or other GitHub documentation.

```markdown
# Advanced Docker Setup for Next.js 14

## Introduction
This guide provides a comprehensive walkthrough for setting up Docker in a Next.js 14 application using advanced techniques. By leveraging Docker, developers can ensure a consistent environment from development to production, enhancing both efficiency and reliability.

## Prerequisites
Before you begin, ensure you have the following installed:
- **Docker**: Download and install Docker from [Docker's official website](https://www.docker.com/products/docker-desktop).
- **Node.js**: Install the latest LTS version from [Node.js official website](https://nodejs.org/).
- A text editor or IDE of your choice.

## Step 1: Setting Up `.dockerignore`
Create a `.dockerignore` file in the root of your Next.js project to exclude unnecessary files from your Docker build context. This helps to reduce build time and image size. Add the following entries:

```plaintext
Dockerfile
.dockerignore
node_modules
npm-debug.log
README.md
.next
.git
```

## Step 2: Configuring `next.config.ts`
Update your `next.config.ts` to enable standalone output, crucial for the Docker setup. Replace its content with the following:

```typescript
/** @type {import('next').NextConfig} */
module.exports = {
  output: "standalone",
};
```

## Step 3: Writing the Dockerfile
Create a `Dockerfile` in your project's root directory. This file will define the multi-stage build process for your Docker image. Here’s the complete Dockerfile:

```Dockerfile
# Base image with Node.js
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm and pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." and exit 1; \
  fi

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN \
  if [ -f yarn.lock ]; then yarn run build; \
  elif [ -f package-lock.json ]; then npm run build; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm and pnpm run build; \
  else echo "Lockfile not found." and exit 1; \
  fi

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
RUN mkdir .next
RUN chown nextjs:nodejs .next
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

## Step 4: Building the Docker Image
Build your Docker image by running the following command in your terminal:

```bash
docker build . -t nextjs-14-docker
```

## Running Your Docker Container
Once the image is built, run your Docker container with:

```bash
docker run -p 3000:3000 nextjs-14-docker
```

This command maps port 3000 of the container to port 3000 on your host, allowing you to access your application at `http://localhost:3000`.

## Conclusion
By following this guide, you have successfully set up a Next.js 14 application in a Dockerized environment using advanced techniques. This setup not only optimizes your development and deployment workflow but also ensures that your application runs consistently across different environments.


Experiment further and integrate more tools into your Docker setup to enhance your development experience with Next.js and Docker.
