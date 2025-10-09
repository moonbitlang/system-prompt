# MoonBit Asynchronous Programming Complete Guide

The MoonBit async library provides comprehensive asynchronous IO functionality and structured concurrency support. This guide is practice-oriented, introducing core concepts and best practices through rich code examples.

## Core Features Overview

- **Structured Concurrency**: Task group management with automatic error propagation and resource cleanup
- **Cooperative Multitasking**: Single-threaded model without locks
- **Comprehensive IO Support**: File system, networking, process, and pipe operations
- **HTTP/TLS Support**: Complete web development capabilities
- **Task Cancellation**: Graceful cancellation mechanisms and timeout control

## Quick Start

### Installation and Setup

```bash
moon add moonbitlang/async@0.7.0
```

**Important**: Add the native target configuration to your project's `moon.mod.json`:

```json
{
  "name": "your-project-name",
  "version": "0.1.0",
  "deps": {
    "moonbitlang/async": "0.8.0"
  },
  "preferred-target": "native"
}
```

The async library currently only supports the native backend. If you encounter compilation errors, try running `moon clean` to switch from the default wasm-gc target to native.

Add required packages to `moon.pkg.json`:

```json
{
  "import": [
    "moonbitlang/async",
    "moonbitlang/async/socket",
    "moonbitlang/async/tls",
    "moonbitlang/async/fs",
    "moonbitlang/async/io",
    "moonbitlang/async/http",
    "moonbitlang/async/process",
    "moonbitlang/async/pipe",
    "moonbitlang/async/aqueue"
  ]
}
```

### Basic Async Program Structure

```moonbit
///|
async fn main {
  println("Hello, Async World!")
  @async.sleep(1000)  // Wait for 1 second
  println("Done!")
}
```

**Key Change**: Now use `async fn main` directly, no longer need `@async.with_event_loop` wrapper.

**Note**: MoonBit doesn't require an `await` keyword. When you call async functions within an `async fn`, the language automatically handles asynchronous execution and suspension points, making the code cleaner and more readable than traditional async/await patterns.

## Asynchronous Operations Basics

### Core Async Functions

```moonbit
///|
async fn basic_operations() -> Unit {
  // Sleep - yield execution to other tasks
  @async.sleep(500)  // 500 milliseconds

  // Pause - actively yield execution
  @async.pause()

  // Get current timestamp
  let now = @async.now()
  println("Current time: \{now}")
}
```

Notice that in MoonBit, there is no `await` keyword, the language will automatically handle the asynchronous execution if the code in this function is asynchronous.

### Timeout Control

```moonbit
///|
async fn timeout_examples() -> Unit {
  // Throw error after timeout
  try {
    @async.with_timeout(1000, fn() {
      @async.sleep(2000)  // Will timeout
      abort("should not reach here")
    })
  } catch {
    @async.TimeoutError => println("Operation timed out")
    // should handle other errors here
    _ => println("Unknown error")
  }

  // Return None after timeout
  let result = @async.with_timeout_opt(1000, fn() {
    @async.sleep(500)
    "Completed successfully"
  })

  match result {
    Some(value) => println("Result: \{value}")
    None => println("Operation timed out")
  }
}
```

## Structured Concurrency and Task Groups

### Task Group Basics

```moonbit
///|
async fn task_group_basics() -> Unit {
  @async.with_task_group(fn(group) {
    // Spawn background task
    group.spawn_bg(fn() {
      @async.sleep(100)
      println("Background task completed")
    })

    // Spawn task with return value
    let task = group.spawn(fn() {
      @async.sleep(200)
      42
    })

    let result = task.wait()
    println("Task result: \{result}")

    "Task group return value"
  }) |> println
}
```

### Task Option Control

```moonbit
///|
async fn task_options() -> Unit {
  @async.with_task_group(fn(group) {
    // Task that's allowed to fail
    group.spawn_bg(allow_failure=true, fn() {
      @async.sleep(50)
      raise Failure("This error won't affect the entire task group")
    })

    // No-wait task - automatically cancelled when task group ends
    group.spawn_bg(no_wait=true, fn() {
      @async.sleep(10000)  // Very long task
      println("This line won't execute")
    })

    @async.sleep(100)
    println("Task group completed normally")
  })
}
```

### Error Propagation Example

```moonbit
///|
async fn error_propagation() -> Unit {
  try {
    @async.with_task_group(fn(group) {
      group.spawn_bg(fn() {
        @async.sleep(100)
        println("Task 1 running normally")
      })

      group.spawn_bg(fn() {
        @async.sleep(50)
        raise Failure("Task 2 failed")  // This will cause the entire task group to fail
      })

      @async.sleep(200)
      println("This line won't execute")
    })
  } catch {
    err => println("Task group failed: \{err}")
  }
}
```

## File System Operations

### File Read/Write

```moonbit
///|
async fn file_operations() -> Unit {
  // Create and write to file
  {
    let file = @fs.create("test.txt", permission=0o644)
    defer file.close()
    file.write("Hello, World!\nSecond line content")
  }

  // Read file
  {
    let file = @fs.open("test.txt", mode=ReadOnly)
    defer file.close()
    // encoding is included in standard library so you can use it directly
    // file.read_all().binary() is Bytes
    // file.read_all().text() is String
    // file.read_all().json() is Json
    let content = @encoding/utf8.decode(file.read_all().binary())
    println("File content: \{content}")
    println("File size: \{file.size()} bytes")
  }

  // Delete file
  @fs.remove("test.txt")
}
```

### Advanced File Operations

```moonbit
///|
async fn advanced_file_ops() -> Unit {
  let file = @fs.open("data.txt", mode=ReadWrite, create=0o644, truncate=true)
  defer file.close()

  // Write data
  file.write_string("Line 1\nLine 2\nLine 3\n",encoding=@io.UTF8)

  // Move file pointer
  let _ = file.seek(0, mode=@fs.FromStart)

  // Chunked reading
  let buf = FixedArray::make(1024, b'0')
  while file.read(buf) is n && n > 0 {
    let arr = buf.unsafe_reinterpret_as_bytes()[0:n]
    let _ = arr[:n]
    println("Read \{n} bytes")
  }
}
```

### Directory Operations

```moonbit
///|
async fn directory_operations() -> Unit {
  // Method 1: Using File::as_dir()
  let dir_file = @fs.open(".", mode=ReadOnly)
  let dir = dir_file.as_dir()
  defer dir.close()

  let files = dir.read_all()
  files.sort()

  for file in files {
    println("File: \{file}")
  }
}

///|
async fn directory_operations_advanced() -> Unit {
  // Method 2: Direct directory operations
  let files = @fs.readdir(".", sort=true, include_hidden=false)
  for file in files {
    println("File: \{file}")
  }

  // Create directory
  @fs.mkdir("new_dir", permission=0o755)

  // Remove directory (recursive)
  @fs.rmdir("old_dir", recursive=true)

  // Check file properties
  if @fs.exists("config.txt") {
    println("Config file exists")
  }

  if @fs.can_read("data.txt") {
    println("Can read data file")
  }

  // Get absolute path
  let abs_path = @fs.realpath("./relative/path")
  println("Absolute path: \{abs_path}")
}
```

### File System Utilities

```moonbit
async fn file_system_utilities() -> Unit {
  // suppose example.txt and input.txt exists in this demo
  let path = "example.txt"

  // Check file existence and permissions
  if @fs.exists(path) {
    println("File exists")

    if @fs.can_read(path) {
      println("Can read file")
    }

    if @fs.can_write(path) {
      println("Can write to file")
    }

    if @fs.can_execute(path) {
      println("Can execute file")
    }
  }

  // Get file type
  let file_kind = @fs.kind(path)
  match file_kind {
    Regular => println("Regular file")
    Directory => println("Directory")
    SymLink => println("Symbolic link")
    _ => println("Other file type")
  }

  // Convenient file operations
  let content = @fs.read_file("input.txt")
  @fs.write_file("output.txt", content.binary(), create=0o644)

  // Text file operations with encoding
  // let text = @fs.read_text_file("input.txt", encoding=UTF8)
  let text = @fs.read_file("input.txt").text()
  // @fs.write_text_file("new_text.txt", text, encoding=UTF8, create=0o644)
  @fs.write_file("new_text.txt", text.to_bytes())
}
```

## Network Programming

### TCP Echo Server

```moonbit
///|
async fn tcp_echo_server() -> Unit {
  @async.with_task_group(fn(root) {
    let server = @socket.TCPServer::new(@socket.Addr::parse("127.0.0.1:8080"))
    defer server.close()

    println("TCP server started on port 8080")

    for {
      let (conn, addr) = server.accept()
      println("New connection from: \{addr}")

      root.spawn_bg(allow_failure=true, fn() {
        defer {
          conn.close()
          println("Connection closed")
        }

        // Echo all data
        conn.write_reader(conn)
      })
    }
  })
}
```

### TCP Client

```moonbit
///|
async fn tcp_client() -> Unit {
  let port = 8080
  let conn = @socket.TCP::connect(@socket.Addr::parse("127.0.0.1:\{port}"))
  defer conn.close()

  // Send data
  conn.write("Hello, Server!\n")

  // Read response
  let buf = FixedArray::make(1024, b'0')
  let n = conn.read(buf)
  let response = buf.unsafe_reinterpret_as_bytes()[0:n]
  println("Server response: \{response}")
}
```

### UDP Example

```moonbit
///|
async fn udp_example() -> Unit {
  let sock = @socket.UDP::new()
  defer sock.close()

  sock.bind(@socket.Addr::parse("0.0.0.0:9090"))

  // Send data
  sock.sendto("UDP message", @socket.Addr::parse("127.0.0.1:9091"))

  // Receive data
  let buf = FixedArray::make(1024, b'0')
  let (n, from_addr) = sock.recvfrom(buf)
  let data = buf.unsafe_reinterpret_as_bytes()[0:n]
  println("Received data from \{from_addr}: \{data}")
}
```

## TLS/SSL Support

### TLS Client

```moonbit
///|
async fn tls_client_example() -> Unit {
  // Connect to HTTPS server using connect_to_host (like HTTP client does)
  let tcp_conn = @socket.TCP::connect_to_host("github.com", port=443)
  defer tcp_conn.close()

  // Create TLS connection
  let tls_conn = @tls.TLS::client(
    tcp_conn,
    verify=true,
    host="github.com",
    sni=true
  )
  defer tls_conn.close()

  // Send HTTP request over TLS
  tls_conn.write("GET / HTTP/1.1\r\nHost: github.com\r\nConnection: close\r\n\r\n")

  // Read response in chunks
  let buf = FixedArray::make(4096, b'0')
  let total_response = StringBuilder::new()

  while tls_conn.read(buf) is n && n > 0 {
    let chunk = buf.unsafe_reinterpret_as_bytes()[0:n]
    total_response.write_string(chunk.to_string())
  }

  println("HTTPS Response: \{total_response.to_string()}")

  // Graceful TLS shutdown
  tls_conn.shutdown()
}
```

### TLS Server (Testing Only)

```moonbit
///|
async fn tls_server_example() -> Unit {
  // WARNING: This API is for testing only
  let tcp_server = @socket.TCPServer::new(@socket.Addr::parse("0.0.0.0:8443"))
  defer tcp_server.close()

  @async.with_task_group(fn(group) {
    for {
      let (tcp_conn, addr) = tcp_server.accept()
      println("TLS connection from: \{addr}")

      group.spawn_bg(allow_failure=true, fn() {
        defer tcp_conn.close()

        let tls_conn = @tls.TLS::server(
          tcp_conn,
          private_key_file="server.key",
          private_key_type=PEM,
          certificate_file="server.crt",
          certificate_type=PEM
        )

        // Handle TLS connection
        // handle_tls_request(tls_conn)

        @async.with_timeout_opt(5000, fn() {
            tls_conn.shutdown()
          }) |> ignore
          tls_conn.close()
      })
    }
  })
}
```

Notice that async function is not supported in `defer` yet.

## HTTP Programming

### HTTP File Server

```moonbit
///|
async fn serve_file(
  conn : @http.ServerConnection,
  file : @fs.File,
  path~ : String,
) -> Unit {
  let content_type = match path {
    [.., .. ".html"] => "text/html"
    [.., .. ".css"] => "text/css"
    [.., .. ".js"] => "text/javascript"
    [.., .. ".png"] => "image/png"
    [.., .. ".jpg"] => "image/jpeg"
    _ => "application/octet-stream"
  }

  conn
  ..send_response(200, "OK", extra_headers={ "Content-Type": content_type })
  ..write_reader(file)
  ..end_response()
}

///|
async fn serve_directory(
  conn : @http.ServerConnection,
  dir : @fs.Directory,
  path~ : String,
) -> Unit {
  let files = dir.read_all()
  files.sort()
  conn
  ..send_response(200, "OK", extra_headers={ "Content-Type": "text/html" })
  ..write("<!DOCTYPE html><html><head></head><body>")
  ..write_string("<h1>\{path}</h1>\n", encoding=UTF8)
  ..write("<div style=\"margin: 1em; font-size: 15pt\">")
  ..write_string(
    "<a href=\"\{path}?download_zip\">download as zip</a><br/><br/>\n",
    encoding=UTF8,
  )
  if path[:-1].rev_find("/") is Some(index) {
    let parent : StringView = if index == 0 { "/" } else { path[:index] }
    conn.write_string("<a href=\"\{parent}\">..</a><br/><br/>\n", encoding=UTF8)
  }
  for file in files {
    let sep = if path[path.length() - 1] != '/' { "/" } else { "" }
    conn.write_string(
      "<a href=\"\{path}\{sep}\{file}\">\{file}</a><br/>\n",
      encoding=UTF8,
    )
  }
  conn..write("</div></body></html>")..end_response()
}

///|
async fn http_file_server() -> Unit {
  let base_path = "."
  let server = @socket.TCPServer::new(@socket.Addr::parse("0.0.0.0:8111"))
  defer server.close()

  println("HTTP file server started on port 8111")

  @async.with_task_group(fn(ctx) {
    for {
      let (tcp_conn, addr) = server.accept()
      let conn = @http.ServerConnection::new(tcp_conn)
      println("New connection from: \{addr}")

      ctx.spawn_bg(allow_failure=true, fn() {
        defer conn.close()

        for {
          let request = conn.read_request()

          match request.meth {
            Get => {
              let file = @fs.open(base_path + request.path, mode=ReadOnly) catch {
                _ => {
                  conn
                  ..send_response(404, "Not Found", extra_headers={ "Content-Type": "text/html" })
                  ..write("<h1>404 - File Not Found</h1>")
                  ..end_response()
                  continue
                }
              }
              defer file.close()

              if file.kind() is Directory {
                serve_directory(conn, file.as_dir(), path=request.path)
              } else {
                serve_file(conn, file, path=request.path)
              }
            }
            _ => {
              conn
              ..send_response(405, "Method Not Allowed")
              ..end_response()
            }
          }
        }
      })
    }
  })
}
```

### HTTP Client Requests

```moonbit
///|
async fn http_client_example() -> Unit {
  // GET request
  let (response, body) = @http.get("https://httpbin.org/get")
  println("Status code: \{response.code}")
  println("Response body: \{body.text()}")

  // POST request
  let post_data = "{ \"key\": \"value\" }"
  let (response, body) = @http.post(
    "https://httpbin.org/post",
    post_data,
    headers={
      "Content-Type": "application/json",
      "User-Agent": "MoonBit-HTTP/1.0"
    }
  )
  println("POST response: \{response.code}")
}
```

## Process Management

### Basic Process Operations

```moonbit
///|
async fn process_operations() -> Unit {
  // Run simple command
  let exit_code = @process.run("ls", ["-la"])
  println("Command exit code: \{exit_code}")

  // Command with environment variables
  @process.run(
    "env",
    [],
    extra_env={ "MY_VAR": "my_value" },
    inherit_env=true
  ) |> ignore
}
```

### Inter-Process Communication

```moonbit
///|
async fn process_communication() -> Unit {
  @async.with_task_group(fn(group) {
    let (cat_read, we_write) = @process.write_to_process()
    let (we_read, cat_write) = @process.read_from_process()
    defer we_read.close()

    // Start process
    group.spawn_bg(fn() {
      @process.run("cat", ["-"], stdout=cat_write, stdin=cat_read)
      |> ignore
    })

    // Send data to process and read results
    group.spawn_bg(fn() {
      we_write.write("async programming\nsync programming\nasync example\n")
      we_write.close()
    })

    // Read process output
    let buf = FixedArray::make(1024, b'0')
    while we_read.read(buf) is n && n > 0 {
      let output = buf.unsafe_reinterpret_as_bytes()[0:n]
      println("Process output: \{output}")
    }
  })
}
```

## Advanced Features

### Retry Mechanisms

```moonbit
///|
async fn retry_examples() -> Unit {
  // Immediate retry
  let result1 = @async.retry(Immediate, max_retry=3, fn() {
    if @async.now() % 2 == 0 {
      raise Failure("Random failure")
    }
    "Success"
  })

  // Fixed delay retry
  let result2 = @async.retry(FixedDelay(1000), max_retry=5, fn() {
    println("Attempting network request...")
    // Simulate network request
    @async.sleep(100)
    "Network request successful"
  })

  // Exponential backoff retry
  let result3 = @async.retry(
    ExponentialDelay(initial=100, factor=2.0, maximum=5000),
    max_retry=10,
    fn() {
      println("Attempting to connect to service...")
      @async.sleep(50)
      "Connection successful"
    }
  )
}
```

### Cancellation Protection

```moonbit
///|
async fn critical_operations() -> Unit {
  @async.with_timeout(100, fn() {
    @async.protect_from_cancel(fn() {
      // This operation won't be interrupted by cancellation
      println("Starting critical operation...")
      @async.sleep(200)  // Will complete even if timeout occurs
      println("Critical operation completed")
    })
  }) catch {
    @async.TimeoutError => println("Operation timed out, but critical part completed")
    _ => println("Unknown error")
  }
}
```

### Task Group Cleanup

```moonbit
///|
async fn task_group_cleanup() -> Unit {
  @async.with_task_group(fn(group) {
    // you can use group.add_defer to clean up resources instead using
    // defer directly
    group.add_defer(fn() {
      println("Executing cleanup operations")
      // Clean up resources
    })

    group.spawn_bg(fn() {
      @async.sleep(100)
      raise Failure("Task failed")  // Trigger cleanup
    })

    @async.sleep(200)
  }) catch {
    err => println("Task group failed, cleanup executed: \{err}")
  }
}
```

## Utilities and Patterns

### Command Line Argument Processing

```moonbit
///|
async fn example_args() -> Unit {
  let args = @env.args()
  match args {
    [] | [_] => {
      println("Usage: program <command> [args...]")
      println("Commands: server, client, test")
    }
    [_, "server", port] => {
      let port_num = @strconv.parse_int(port) catch { _ => 8080 }
      // start_server(port_num)
      ...
    }
    [_, "client", host, port] => {
      // connect_client(host, port.to_int().unwrap())
      ...
    }
    [_, "test"] => {
      // run_tests()
      ...
    }
    _ => println("Unknown command")
  }
}
```

### Cat Tool Implementation

```moonbit
///|
async fn cat_implementation() -> Unit {
  let args = @env.args()
  match args {
    [] | [_] => {
      // Read from standard input
      @pipe.stdout.write_reader(@pipe.stdin)
    }
    [_, .. files] => {
      for file in files {
        if file == "-" {
          @pipe.stdout.write_reader(@pipe.stdin)
        } else {
          let file_handle = @fs.open(file, mode=ReadOnly)
          defer file_handle.close()
          @pipe.stdout.write_reader(file_handle)
        }
      }
    }
  }
}
```

This implementation demonstrates several key async programming patterns:
- **Command line argument processing** using pattern matching
- **Resource management** with `defer` for automatic cleanup
- **Efficient IO streaming** using `write_reader` to copy data without buffering entire files
- **Standard input/output handling** through the `@pipe` module
- **Multiple file processing** in a single program

Usage examples:
```bash
# Read from stdin
echo "Hello World" | ./cat

# Read single file
./cat file.txt

# Read multiple files
./cat file1.txt file2.txt file3.txt

# Mix files and stdin
./cat file1.txt - file2.txt  # "-" represents stdin
```



### Async Queue Communication

```moonbit
///|
async fn queue_communication() -> Unit {
  @async.with_task_group(fn(group) {
    let queue = @aqueue.Queue::new()

    // Producer task
    group.spawn_bg(fn() {
      for i in 0..<10 {
        queue.put("Message \{i}")
        @async.sleep(100)
      }
    })

    // Consumer task
    @async.with_timeout(10000, fn() {
      group.spawn_bg(fn() {

        for {
          let msg = queue.get() catch {
            _ => break
          }
          println("Received: \{msg}")
        }
        println("Queue closed")
      })
    })
  })
}
```

### Buffered IO Operations

```moonbit
///|
async fn buffered_io_examples() -> Unit {
  let file = @fs.open("large_file.txt", mode=ReadOnly)
  defer file.close()

  // Buffered reader for efficient reading
  let reader = @io.BufferedReader::new(file)

  // Search for patterns
  let pattern_pos = reader.find("target_pattern")
  println("Pattern found at position: \{pattern_pos}")

  // Get specific bytes without advancing stream
  let first_byte = reader[0]
  let slice = reader[10:20]

  // Drop unwanted data
  reader.drop(100)

  // Continue reading normally
  let remaining = reader.read_all()
  println("Remaining content: \{remaining.text()}")
}

///|
async fn buffered_writer_example() -> Unit {
  let file = @fs.create("output.txt", permission=0o644)
  defer file.close()

  let writer = @io.BufferedWriter::new(file)

  // Efficient writing with buffering
  for i in 0..<1000 {
    writer.write("Line \{i}\n")
  }

  // Manual flush if needed
  writer.flush()
}
```

## Performance Optimization and Best Practices

### 1. Resource Management
```moonbit
///|
async fn resource_management() -> Unit {
  // Always use defer to ensure resource cleanup
  let file = @fs.open("data.txt", mode=ReadOnly)
  defer file.close()

  let conn = @socket.TCP::new()
  defer conn.close()

  // Use file and connection...
}
```

### 2. Error Handling Strategies
```moonbit
///|
async fn error_handling_patterns() -> Unit {
  @async.with_task_group(fn(group) {
    // Critical task - entire task group fails if this fails
    group.spawn_bg(fn() {
      // critical_operation()
      ...
    })

    // Optional task - failure doesn't affect other tasks
    group.spawn_bg(allow_failure=true, fn() {
      // optional_operation()
      ...
    })

    // Background task - automatically cancelled when task group ends
    group.spawn_bg(no_wait=true, fn() {
      // background_monitoring()
      ...
    })
  })
}
```

### 3. Concurrency Control
```moonbit
///|
async fn concurrency_patterns() -> Unit {
  // Limit concurrency
  @async.with_task_group(fn(group) {
    let semaphore = @async.Queue::new()

    // Initialize semaphore
    for _ in 0..<5 {
      semaphore.put(())
    }

    for i in 0..<20 {
      group.spawn_bg(fn() {
        let _permit = semaphore.get()  // Acquire permit
        defer semaphore.put(())        // Release permit

        // Execute limited concurrency task
        //expensive_operation(i)
      })
    }
  })
}
```

### 4. Yielding During Long Computations
```moonbit
///|
async fn long_computation() -> Unit {
  for i in 0..<1000000 {
    // Perform computation
    // heavy_calculation(i)

    // Periodically yield execution
    if i % 1000 == 0 {
      @async.pause()
    }
  }
}
```

## Common Pitfalls and Considerations

### 1. Avoid Blocking Operations
```moonbit
// ❌ Wrong - blocks entire program
fn blocking_operation() {
  // Long synchronous computation or blocking IO
}

// ✅ Correct - use async operations
async fn non_blocking_operation() {
  // Use @async.sleep, @fs.*, @socket.* and other async APIs
  @async.pause()  // Actively yield execution
}
```

### 2. Proper Error Propagation
```moonbit
// ❌ Wrong - ignoring errors
async fn bad_error_handling() {
  risky_operation() catch { _ => () }  // Silently ignore errors
}

// ✅ Correct - properly handle errors
async fn good_error_handling() {
  risky_operation() catch {
    err => {
      log_error(err)
      // Re-throw or handle as needed
      raise err
    }
  }
}
```

### 3. Task Lifecycle Management
```moonbit
// ❌ Wrong - potential resource leak
async fn bad_task_management() {
  let task = group.spawn(fn() { long_running_task() })
  // Forgot to wait for task completion
}

// ✅ Correct - explicit lifecycle
async fn good_task_management() {
  @async.with_task_group(fn(group) {
    group.spawn_bg(fn() { long_running_task() })
    // Task group ensures all tasks complete
  })
}
```

## Debugging and Testing

### Testing Async Functions

MoonBit provides excellent support for testing async functions using `async test`. Here are common patterns for testing async code:

#### Basic Async Test
```moonbit
///|
async test "basic async operation" {
  let result = @async.with_timeout(1000, fn() {
    @async.sleep(100)
    "completed"
  })

  inspect(result, content="completed")
}
```

#### Testing Task Groups and Concurrency
```moonbit
///|
async test "task group coordination" {
  let log = StringBuilder::new()
  @async.with_task_group(fn(group) {
    // Producer task
    group.spawn_bg(fn() {
      for i in 0..<3 {
        @async.sleep(50)
        log.write_string("produced \{i}\n")
      }
    })

    // Consumer task
    group.spawn_bg(fn() {
      for i in 0..<3 {
        @async.sleep(75)
        log.write_string("consumed \{i}\n")
      }
    })
  })

  // Verify the interleaved execution
  inspect(
    log.to_string(),
    content=(
      #|produced 0
      #|produced 1
      #|consumed 0
      #|produced 2
      #|consumed 1
      #|consumed 2
      #|
    ),
  )
}
```

#### Testing Queue Operations
```moonbit
///|
async test "queue producer-consumer" {
  let log = StringBuilder::new()
  @async.with_task_group(fn(group) {
    let queue = @aqueue.Queue::new()

    // Consumer task
    group.spawn_bg(fn() {
      for _ in 0..<3 {
        let item = queue.get()
        log.write_string("consumed: \{item}\n")
      }
    })

    // Producer task
    group.spawn_bg(fn() {
      for i in 0..<3 {
        @async.sleep(20)
        queue.put(i)
        log.write_string("produced: \{i}\n")
      }
    })
  })
  inspect(
    log.to_string(),
    content=(
      #|produced: 0
      #|consumed: 0
      #|produced: 1
      #|consumed: 1
      #|produced: 2
      #|consumed: 2
      #|
    ),
  )
}
```

#### Testing Non-blocking Operations
```moonbit
///|
test "synchronous queue operations" {
  let queue = @aqueue.Queue::new()

  // Test empty queue
  inspect(queue.try_get(), content="None")

  // Test with items
  queue.put(42)
  queue.put(100)

  inspect(queue.try_get(), content="Some(42)")
  inspect(queue.try_get(), content="Some(100)")
  inspect(queue.try_get(), content="None")
}
```

#### Testing File Operations
```moonbit
///|
async test "file operations" {
  let test_content = "Hello, async world!"

  // Write test file
  @fs.write_file("test_async.txt", test_content.to_bytes(), create=0o644)

  // Read and verify
  let content = @fs.read_file("test_async.txt")
  inspect(content.text(), content="Hello, async world!")

  // Cleanup
  @fs.remove("test_async.txt")
}
```

#### Testing Network Operations (Mock Example)
```moonbit
///|
async test "network timeout handling" {
  // Test connection timeout
  let result = @async.with_timeout_opt(100, fn() {
    @async.sleep(200)  // Simulate slow network
    "connection established"
  })

  inspect(result, content="None")  // Should timeout

  // Test successful connection
  let result2 = @async.with_timeout_opt(200, fn() {
    @async.sleep(50)   // Fast connection
    "connection established"
  })

  inspect(result2, content="Some(\"connection established\")")
}
```

### Testing Best Practices

1. **Use `inspect()` for Assertions**: MoonBit's `inspect()` function is perfect for testing async results
2. **Test Timing Behavior**: Use `@async.sleep()` to simulate realistic timing scenarios
3. **Test Error Conditions**: Always test timeout, cancellation, and error propagation
4. **Use StringBuilder for Logging**: Capture execution order and verify concurrent behavior
5. **Test Resource Cleanup**: Ensure `defer` and cleanup work correctly
6. **Test Edge Cases**: Empty queues, cancelled operations, failed connections

### Logging
```moonbit
///|
async fn logging_example() -> Unit {
  @pipe.stderr.write("Starting request processing\n")

  try {
    //process_request()
    ...
  } catch {
    err => {
      @pipe.stderr.write("Processing failed: \{err}\n")
      raise err
    }
  }

  @pipe.stderr.write("Request processing completed\n")
}
```

### Performance Measurement
```moonbit
///|
async fn performance_measurement() -> Unit {
  let start = @async.now()

  // expensive_operation()
  println("===")

  let duration = @async.now() - start
  println("Operation took: \{duration} milliseconds")
}
```

## Summary

The MoonBit async library provides powerful and safe asynchronous programming capabilities through structured concurrency:

- **Simple**: Use `async fn main` directly, no complex event loop setup required
- **Safe**: Structured concurrency ensures no resource leaks and proper error propagation
- **Efficient**: Cooperative multitasking model without locks
- **Complete**: Covers all IO needs including files, networking, HTTP, TLS, and processes

Mastering task group usage patterns and error handling mechanisms is key to successfully using this library. Through proper task decomposition and resource management, you can build high-performance, maintainable asynchronous applications.
