FROM golang:1.19 AS build-stage
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /docker-gs-ping

FROM gcr.io/distroless/base-debian11 AS build-stage-2
WORKDIR /
COPY --from=build-stage /docker-gs-ping /docker-gs-ping
EXPOSE 8080
ENTRYPOINT ["/docker-gs-ping"]