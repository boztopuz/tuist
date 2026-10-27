# SwifterPM

Use the shared `fileSystem` instance from `tuist/FileSystem` for filesystem
operations. Convert URLs to `AbsolutePath` with the in-module `URL.absolutePath`
helper. Do not use `Foundation.FileManager`, NIOFileSystem, or
`swift-tools-support-core` filesystem primitives directly.
