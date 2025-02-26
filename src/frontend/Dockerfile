# Etapa de construcción
FROM --platform=$BUILDPLATFORM golang:1.23.4-alpine@sha256:c23339199a08b0e12032856908589a6d41a0dab141b8b3b21f156fc571a3f1d3 AS builder
ARG TARGETOS
ARG TARGETARCH
WORKDIR /src/frontend

# Instala git (necesario para descargar dependencias de repositorios Git)
RUN apk add --no-cache git

# Copia los archivos de dependencias
COPY src/frontend/go.mod src/frontend/go.sum ./

# Copia los módulos locales
COPY src/frontend/genproto ./genproto
COPY src/frontend/money ./money
COPY src/frontend/validator ./validator

# Descarga las dependencias
RUN go mod download

# Copia todo el código fuente
COPY src/frontend/ ./

# Restaura dependencias y limpia módulos innecesarios
RUN go mod tidy

# Compila la aplicación
ARG SKAFFOLD_GO_GCFLAGS
RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} CGO_ENABLED=0 go build -gcflags="${SKAFFOLD_GO_GCFLAGS}" -o /go/bin/frontend .

# Etapa final
FROM scratch
WORKDIR /src
COPY --from=builder /go/bin/frontend /src/server
COPY src/frontend/static ./static
COPY src/frontend/templates ./templates

ENV GOTRACEBACK=single
EXPOSE 8080
ENTRYPOINT ["/src/server"]