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

### Commit 3 Reflection Notes

In the initial stage, the website responds to each client request by sending the same HTML content, `hello.html`, regardless of what the client requests. This means that no matter what path the client requests, the server always sends the same HTML.

1. **Creating an Alternative HTML Content for Unavailable Paths**

Initially, I created a `404.html` file to serve as alternative content if the requested path is not available on the server. The content of `404.html` is as follows:
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Hello!</title>
    </head>
    <body>
        <h1>Oops!</h1>
        <p>Sorry, I don't know what you're asking for.</p>
        <p>Rust is running from ian's machine.</p>
    </body>
    </html>
    ```

2. **Modifying the `handle_connection` Function**
In the `src\main.rs` file, there are changes made to the handle_connection function to handle responses. The function now looks like this:
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```
In this refactored version:
- **Duplication of Code Elimination**: The code block for reading files and sending responses was duplicated inside the if and else blocks. This inefficiency increases the risk of errors when changes are made.
- **Improved Code Readability**: By separating the logic to determine the status line and file name from the logic to read files and send responses, the code becomes easier to read.
- **Facilitation of Future Modifications**: If there is a need to change how the server reads files or sends responses, such changes only need to be made once. This greatly assists in code maintenance and the development of new features.
- **Conditional Selection**: Based on the request line, a tuple is used to select the appropriate status line and filename. Then, the server reads the content of the relevant file and sends the response accordingly. This makes the code more modular and easier to manage.

Refactoring in this manner enhances the efficiency, readability, and maintainability of the codebase.

- Screenshot of Proof
![Commit 3 screen capture](/assets/images/commit3failed.png)

### Commit 4 Reflection Notes

In the updated `handle_connection` function, there are slight changes where the if-else part is replaced with a `match` statement. The `match` statement is used to match the value of `request_line` with several predefined patterns: `/`, `/sleep`, and others. The introduction of a new endpoint, `/sleep`, demonstrates how a single-threaded server can handle requests that require longer processing times.

In this endpoint, `thread::sleep(Duration::from_secs(10));` is executed to delay processing for 10 seconds. However, when there are multiple users, this can be highly disruptive because several seconds can be crucial. In the simulated slow response scenario, when a user opens two browser windows and tries to access `127.0.0.1/sleep` and `127.0.0.1` in the other window, the user must wait for requests to be processed sequentially due to the single-threaded nature.

When a request to `/sleep` is processed, it illustrates how a website that needs to respond within a few seconds can cause delays when responding to other requests. Thus, the use of a single thread can consume time by potentially blocking the processing of other requests on the server.

### Commit 5 Reflection Notes

To implement an efficient system using multithreading, we need to build a `ThreadPool` that allows us to handle various requests simultaneously. The next step involves creating `Workers`, which are responsible for receiving and executing specific tasks or jobs sent to each request. To ensure smooth communication between the `ThreadPool` and each `Worker`, we need to set up a message passing mechanism, where the `ThreadPool` has the ability to send signals via a sender to cloned receivers assigned to each Worker. This ensures that when the `ThreadPool` receives a request to execute a task, a signal will be sent via the sender to the relevant `Worker`'s receiver, which then processes the job.

Within each `Worker`, there is one thread consistently waiting for incoming data. As soon as the data is received, the `Worker` will lock the receiver to process that data. Once the process is completed, the lock on the receiver will be released, allowing another `Worker` to receive information and execute the next task.

### Commit 6 Reflection Notes
[The Rust Programming Language Book](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html) suggests using the `build` method instead of `new` when initializing the `ThreadPool`, arguing that `new` may cause an error if the number of threads given is too small. However, this argument is incorrect because the expectation of the `new` method is success. Therefore, it is recommended to replace the `new` method with `build`, which returns `Result`. Then, when the value is returned to the caller, it can be unwrapped to get the value of the execution result.