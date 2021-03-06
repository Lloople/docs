# Files

Vapor offers a simple API for reading and writing files asynchronously within route handlers. This API is built on top of NIO's [`NonBlockingFileIO`](https://apple.github.io/swift-nio/docs/current/NIO/Structs/NonBlockingFileIO.html) type.

## Read

The main method for reading a file delivers chunks to a callback handler as they are read off the disk. The file to read is specified by its path. Relative paths will look in the process's current working directory.

```swift
// Asynchronously reads a file from disk.
req.fileio.readFile(at: "/path/to/file") { chunk in
    print(chunk) // ByteBuffer
}
```

The returned future will signal when the read has completed or an error has occured.

### Stream

The `streamFile` method converts a streaming file to a `Response`. This method will set appropriate headers such as `ETag` and `Content-Type` automatically.

```swift
// Asynchronously streams file as HTTP response.
req.fileio.streamFile(at: "/path/to/file").map { res in
    print(res) // Response
}
```

The result can be returned directly by your route handler. 

### Collect 

The `collectFile` method reads the specified file into a buffer.

```swift
// Reads the file into a buffer.
req.fileio.collectFile(at: "/path/to/file").map { buffer in 
    print(buffer) // ByteBuffer
}
```

!!! warning
    This method requires the entire file to be in memory at once. Use chunked or streaming read to limit memory usage.

## Write

The `writeFile` method supports writing a buffer to a file.

```swift
// Writes buffer to file.
req.fileio.writeFile(ByteBuffer(string: "Hello, world"), at: "/path/to/file")
```

The returned future will signal when the write has completed or an error has occured.

## Middleware

For more information on serving files from your project's _Public_ folder automatically, see [Middleware &rarr; FileMiddleware](middleware.md#file-middleware).

## Advanced

For cases that Vapor's API doesn't support, you can use NIO's `NonBlockingFileIO` type directly. 

```swift
// Main thread.
let fileHandle = try app.fileio.openFile(
    path: "/path/to/file", 
    eventLoop: app.eventLoopGroup.next()
).wait()
print(fileHandle)

// In a route handler.
req.application.fileio.openFile(
    path: "/path/to/file", 
    eventLoop: req.eventLoop
).map { fileHandle in
    print(fileHandle)
}
```

For more information, visit SwiftNIO's [API reference](https://apple.github.io/swift-nio/docs/current/NIO/Structs/NonBlockingFileIO.html).
