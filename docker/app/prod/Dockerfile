FROM golang:alpine AS builder

ENV USER=appuser
ENV UID=10001

RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid "${UID}" \
    "${USER}"

WORKDIR /app

COPY go.mod ./

ENV GO111MODULE=on
RUN go mod download
RUN go mod verify

COPY . ./

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' -a \
    -o antifraud-app ./cmd/app/main.go

FROM scratch

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group
COPY --from=builder /app/antifraud-app antifraud-app

COPY configs/ configs/
COPY .env .env

USER appuser:appuser

ENTRYPOINT ["./antifraud-app"]
