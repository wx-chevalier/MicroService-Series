# gRPC

gRPC 是由 Google 开源的 RPC 框架，其内置了高性能的二进制编码，相较于 JSON 与 HTTP 其有更好的压缩率。当我们开发 gRPC 接口时，首先要定义一个 .proto 文件：

```go
syntax = "proto3";

package fromatob;

// FromAtoB is a simplified version of fromAtoB’s backend API.
service FromAtoB {
	rpc Lookup(LookupRequest) returns (Coordinate) {}
}

// A LookupRequest is a request to look up the coordinates for a city by name.
message LookupRequest {
	string name = 1;
}

// A Coordinate identifies a location on Earth by latitude and longitude.
message Coordinate {
	// Latitude is the degrees latitude of the location, in the range [-90, 90].
	double latitude = 1;

	// Longitude is the degrees longitude of the location, in the range [-180, 180].
	double longitude = 2;
}
```

然后我们可以利用 protoc 工具从该接口文件中生成服务端与客户端代码。

# 开发环境

## protoc

1. Download the appropriate release here: https://github.com/google/protobuf/releases
2. Unzip the folder
3. Enter the folder and run `./autogen.sh && ./configure && make`
4. If you run into this error: _autoreconf: failed to run aclocal: No such file or directory,_ run `brew install autoconf && brew install automake.` And run the command from step 3 again.
5. Then run these other commands. They should run without issues

```sh
$ make check

$ sudo make install

$ which protoc

$ protoc --version
```