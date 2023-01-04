FROM golang:buster as build

WORKDIR /go/src/grpc-go

COPY . .

WORKDIR /go/src/grpc-go/examples/route_guide/server

RUN go build -o route_guide server.go

FROM gcr.io/distroless/base-debian10

COPY --from=build /go/src/grpc-go/examples/route_guide/server/route_guide /

EXPOSE 50051

CMD ["/route_guide"]
