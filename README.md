# Module 6
## Reflection

### Commit 1 Reflection Notes

### 1. main
1. TCP Listener Creation:
```rust
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
```
- A TCP listener is created using `TcpListener::bind()`. This binds the listener to the address `127.0.0.1:7878`, indicating that it should listen for incoming TCP connections on port `7878`.
- The `unwrap()` method is called to handle any potential errors. If the binding operation fails (e.g., if the port is already in use), the program will panic.

2. Incoming Connection Handling
```rust
for stream in listener.incoming() {
    let stream = stream.unwrap();

    handle_connection(stream);
}
```
- The program enters a loop to handle incoming TCP connections. The `incoming()` method of the `TcpListener` returns an iterator over incoming connections.
- For each incoming connection, the stream is unwrapped from the `Option<TcpStream>`. If there's an error accepting the connection, the program will panic.
- The `handle_connection` function is called with the accepted `TcpStream` to process the incoming connection.

### 2. handle_connection
1. Function Signature
```rust
fn handle_connection(mut stream: TcpStream)
```
- The function takes a `TcpStream` as its argument, representing the connection to a client.
- A `BufReader` is created, wrapping the TCP stream. This allows efficient reading of data from the stream by buffering input and reducing the number of system calls.

2. Buffered Reader Creation
```rust
let buf_reader = BufReader::new(&mut stream);
```
- A `BufReader` is created, wrapping the TCP stream. This allows efficient reading of data from the stream by buffering input and reducing the number of system calls.

3. Reading HTTP Request
```rust
let http_request: Vec<_> = buf_reader
    .lines()
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty())
    .collect();
```
- The `lines()` method of `BufReader` is used to obtain an iterator over the lines of input from the TCP stream.
- The `map()` method is used to unwrap each `Result` returned by the iterator, converting it to a String.
- The `take_while()` method is used to collect lines until an empty line is encountered, signifying the end of the HTTP request headers.
- The collected lines are stored in a `Vec<String>` named http_request.

4. Print HTTP Request
```rust
println!("Request: {:#?}", http_request);
```
- Finally, the function prints the collected HTTP request data to the console.

### Commit 2 Reflection Notes

![Commit 2 screen capture](/assets/images/commit2.png)


In this commit, there are several additions to the `handle_connection` function. The function is now not only capable of reading requests through a TCP stream but also able to respond to them. Its functionality is expanded by sending an HTTP response to the client containing the content of `hello.html`. This process is achieved by creating a response status line indicating that the request has been successfully processed with the code "200 OK". Then, the content of the `hello.html` file is read, and its length is calculated to be included in the response header. The complete HTTP response, including the status line, headers, and content, is then constructed and sent back to the client through the same stream. This illustrates how a basic web server can respond to HTTP requests by sending web page content to the client that sent the request.