FROM golang:1.17-alpine AS builder

WORKDIR /app

RUN apk add --no-cache protobuf

# Copy go mod and sum files
COPY go.mod go.sum ./

# Download all dependencies
RUN go mod download

RUN go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.27.1
RUN go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1.0

# Copy the source code
COPY . .

RUN protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative proto/session.proto

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Use a smaller image for the final stage
FROM alpine:latest  

WORKDIR /root/

# Copy the binary from builder
COPY --from=builder /app/main .
COPY --from=builder /app/proto ./proto

# Expose the gRPC port
EXPOSE 50051

# Command to run
CMD ["./main"]
