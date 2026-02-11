🐳 Multi-Stage Dockerfile — Detailed Notes
1️⃣ What is a Multi-Stage Dockerfile?

A multi-stage Dockerfile uses multiple FROM instructions to separate the build environment and the runtime environment, and copies only the final required output into the final image.

Instead of shipping everything:

code + compilers + build tools + dependencies + runtime


We ship only:

runtime + final application

2️⃣ Why Multi-Stage Build Exists (The Real Problem)

Normal Dockerfile:

FROM node:20
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["npm","start"]


This image contains:

source code

node_modules

build tools

package manager

cache files

development dependencies

Result:

Large image
Slow start
Security risk
Not production ready


But production only needs:

compiled application
runtime


So we separate build and run.

3️⃣ Core Idea

Build in one container → Run in another clean container

4️⃣ Basic Structure
# Stage 1 (builder)
FROM node AS build
RUN build application

# Stage 2 (runtime)
FROM nginx
COPY --from=build /output /app


Each FROM starts a new independent image.

5️⃣ How Docker Executes Multi-Stage Build

Step-by-step:

Docker builds first stage (temporary image)

Runs commands

Produces build output

Starts fresh new image

Copies selected files

Discards builder stage

Important:

Builder image is NOT shipped to production

6️⃣ What Actually Gets Copied

Only files you specify:

COPY --from=build /app/dist /usr/share/nginx/html


NOT copied:

OS packages

compilers

cache

temp files

secrets

So runtime image stays minimal.

7️⃣ Example — React Application
# Build stage
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# Runtime stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html


Final image contains:

nginx + static files only


No node, no npm.

8️⃣ Example — Python Application
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

FROM python:3.11-slim
COPY --from=builder /install /usr/local
COPY . .
CMD ["python","app.py"]


Final image contains only runtime dependencies.

9️⃣ Example — Java Application
FROM maven AS build
COPY . .
RUN mvn package

FROM openjdk:17-jre
COPY --from=build target/app.jar app.jar
CMD ["java","-jar","app.jar"]


Build with JDK → Run with JRE.

🔟 Key Concept — Builder vs Runtime
Builder Stage	Runtime Stage
Temporary	Final image
Heavy	Lightweight
Has tools	Minimal tools
Used to prepare app	Used to run app
1️⃣1️⃣ Advantages of Multi-Stage Builds
Smaller Images

Removes unnecessary files → faster pull and deploy

Improved Security

Removes:

compilers

shells

package managers

debug tools

Reduces attack surface.

Faster Deployment

Small images start faster and scale faster.

Clean Environment

No development tools in production.

Immutable Infrastructure

Containers cannot modify themselves easily.

Better CI/CD Performance

Cached build layers → faster builds.

Protects Source Code

Only compiled output copied.

1️⃣2️⃣ Security Benefits Explained

Without multi-stage attacker can:

install malware
compile exploits
read source code
modify app


With multi-stage attacker only sees runtime.

No tools → attack difficult.

1️⃣3️⃣ Multi-Stage Build Patterns
Build → Serve (Frontend)

Node → Nginx

Compile → Run (Backend)

Maven → JRE

Install → Execute (Python)

Builder → Slim runtime

Package → Deploy (Go)

Go build → scratch image

1️⃣4️⃣ Naming Stages
FROM node AS builder
FROM nginx AS runtime


Used in copy:

COPY --from=builder /app/build /usr/share/nginx/html


Improves readability.

1️⃣5️⃣ Using Multiple Builders

You can chain multiple build stages:

FROM node AS deps
FROM node AS build
FROM nginx AS runtime

1️⃣6️⃣ What Happens to Previous Stages?

They exist only during build.

After image is built:

only last stage survives

1️⃣7️⃣ Important Rules

Each FROM resets filesystem

Nothing carries automatically

Only COPY --from transfers files

1️⃣8️⃣ Best Practices
Always Use Minimal Runtime Image
alpine / slim / jre / scratch

Do Not Copy Entire Project

Copy only required output.

Use Named Stages

Avoid index numbers.

Add USER in Final Stage

Security requirement.

1️⃣9️⃣ Common Mistakes

❌ Running application in builder stage
❌ Copying unnecessary files
❌ Using full OS in runtime
❌ Forgetting .dockerignore

2️⃣0️⃣ When NOT Needed

Simple scripts:

single python file
simple static nginx


Multi-stage useful mainly for build processes.
