# Parallel HTTP proxy with caching 

# Intro 

HTTP proxies have a lot of use cases, from caching, authentication and preventing users from accessing malicious websites. They’re widely used in the world wide web and in business settings. Businesses use proxies to prevent their employees from surfing malicious websites, and enforce authentication and logging. Websites use proxies to provide caching and load balancing that shrinks websites load time.

# Requirements

You’re asked to implement a parallel HTTP proxy server that accepts a GET request and makes it on behalf of the client. The HTTP proxy returns the response if succeeded to the client, and an 404 if there was an error. 
The proxy DOES handle multiple clients in parallel, more on that later. It  handles GET requests in parallel.
Other request methods (PUT, PATCH, DELETE, ...etc) received by the proxy should return a 501 not implemented error. 
You don’t need to read any RFCs for this project, but skimming the HTTP 1.0 RFC will be useful for you.
Reading the project description on Princeton’s website is recommended.
Before you start, an introduction to HTTP
Read this section from the original project spec. but try those steps on your computer
telnet info.cern.ch 80
	Now, this will open a TCP connection to this website on port 80. You should see something like
Trying 188.184.64.53...
Connected to info.cern.ch.
Escape character is '^]'. 
	Now you're connected to this website, to make a GET request type in (note: the host header is required to get a correct answer)
GET /hypertext/WWW/TheProject.html HTTP/1.1
Host: info.cern.ch
	You’ll notice that you get back an HTML response. Now you’ve fetched the first website that ever existed on the internet, that was all the internet back then! You can also use curl (usage - proxy config) instead of telnet.
Starting up
After starting the proxy, it will wait for an incoming request using TCP connections. Upon receiving a HTTP request. The proxy parses the request and verifies that it’s valid. An  invalid request would return a bad request with status code 400 to the client. 
Requirement Checklist
Please do the following in order. To make things easy for you, it’s fine if you amend a piece of your code when moving from a step to another, that’s much much better than for example trying to make two points at once, please move one step at a time.
> Make sure your code validates HTTP requests correctly [20%]
This part has nothing to do with sockets, so it’s the best thing to start with since it has no dependencies. You should consider this part complete when your code is able to extract the HTTP method, headers, target host, and path. A valid HTTP request contains the following parts
<METHOD> <URL or PATH> <HTTP VERSION>
And a Host header, if the specified resource is a PATH (relative):
Host: <HOSTNAME>
All other headers just need to be properly formatted [name] [colon] [value], any names and values are valid.
<HEADER NAME>: <HEADER VALUE>
After parsing HTTP requests correctly, make sure you can identify which requests are malformed (have missing required parts or bad method). For HTTP/1.0 all the available methods are GET, HEAD, POST, -PUT- (since it was in the examples) any other method will return 501, missing parts in the request line of malformed headers will return 400.
Forming the correct corresponding error object HttpErrorResponse and serializing it to a valid HTTP response (HTTP string).
Steps (if you need more guidance)
   * Take out the first line in the request
   * Split it using " " (space)
   * Make sure it has only 3 non-empty components
   * Check HTTP version exists -> A string that starts with "HTTP/"
   * Check than a path exists -> Path isn't empty
   * Check than a method exists -> Method isn't empty
   * If any of the above isn't satisfied, return INVALID_REQUEST
   * Check that the method is valid
   * Valid methods are GET, HEAD, POST, PUT
   * If it's not the case, return INVALID_REQUEST
   * Check that the method is GET
   * If it's not the case, return NOT_IMPLEMENTED
   * Check the headers if any
   * Take all other lines in the request
   * Make sure that each of them contains a ":"
   * Make sure splitting by ":" returns only 2 non-empty components
   * Return a list of [ <HEADERNAME>,<HEADERVALUE> ] (list of lists)
   * If any of the above fails, return INVALID_REQUEST
   * If all that goes well, return GOOD
   * You shouldn't check the character casing (upper / lower case), so you can simply lowercase() the request then do the validation
   * If the flow fails at any point, just return an INVALID_REQUEST if nothing is specified. Never check for failing cases as they're infinite.


> Make sure your code parses the URL correctly (i.e. Sanitization)
After validation, the proxy will need to parse the requested URL. The proxy needs at most three pieces of information: the requested host and port, and the requested path. If parsing the URL fails, return INVALID_REQUEST, don't validate the URL.
________________


Steps (how to extract host, port, path)
You already have the requested path from the previous step
   * if it's a relative path already
   * Get the host and port from the Host header
   * Congrats, now you have the host (from header), path and port
   * If it's an absolute path
   * Check if the URL starts with "http", if so, split it by "://" to get the website URL (host + port)
   * If not, you already have the website URL (host + port)
   * Extract the port if it exists (now you have the website URL (host+port))
   * Check if ":" is present
   * If so, get the port number (you don't need to check if it's already a number)
   * Otherwise, use the default port: 80
   * Extract the host and path, split the full URL by "/" and get the first component -> host, the remaining of the string is the path.
   * If no splits were returned, the requested path is the root "/"
   * If any step fails, return INVALID_REQUEST
After that put the request on the standard form, using relative url + host header.
Examples
Host Extraction
GET www.google.com HTTP/1.0\r\nِ\r\n
	The path will be / and the target host will be www.google.com also because there’s no host header.
If the request had a relative URL, like this
GET / HTTP/1.0\r\nHost: www.google.com\r\n\r\n
	The path will be / and the host will be www.google.com, obtained from the Host: header, which will be the first header. You can assume that either one of the two cases will always happen.
Port Extraction
To extract the port number you’ll find it if a colon (“:”) exists in the host (from the host header) or in the absolute URL like so:
GET www.google.com:8080/things HTTP/1.0\r\n\r\n
	Or
GET /things HTTP/1.0\r\nHost: www.google.com:8080\r\nAccept: application/json\r\n\r\n
	You’ll find a separate function in the skeleton that takes an HTTP string and should return an HttpRequestInfo object.
> Make sure your code accepts a single TCP connection correctly and rejects bad requests 
After your parsed and verified HTTP requests, now you’re ready to use sockets, our first goal is to make sure everything works fine with a single client sending one request per time. To check your code validity; print the address of the client when you call accept() on the server socket and then try sending back anything to this client, so you can know how to use the different mechanism TCP sockets work with.
Now try sending a bad requests like
   * PUT www.google.com/ HTTP/1.0\r\n\r\n gets a "Not Implemented" (501) response.
   * GOAT www.google.com/ HTTP/1.0\r\n\r\n gets a “Bad Request” (400) response.
For HTTP/1.0 all the available methods are GET, HEAD, POST, -PUT- (since it was in the examples) any other method will return 501
> Make sure you can act as a proxy for a single client sending sequential requests 
Once the proxy has parsed the URL, it can make a connection to the requested host (using the appropriate remote port, or the default of 80 if none is specified) and send the HTTP request for the appropriate resource. The proxy should always send the request in the relative URL + Host header format regardless of how the request was received from the client:
Accept from client:
GET http://eng.alexu.edu.eg/ HTTP/1.0
	Or 
GET / HTTP/1.0
Host: eng.alexu.edu.eg
	(note that the Host header is required above since the GET had a relative path)
________________


Send to remote server:
GET / HTTP/1.0
Host: eng.alexu.edu.eg
(Additional client specified headers, if any...)
	

Again, no threading yet. After you accept the client’s connection, read their HTTP request -> sanitize it -> reject it if invalid -> send it to the remote server and then the new part is to read the response of the remote server then send it back to the client socket and close their connection.
You should be able to return results to clients requesting HTTP websites like the first website correctly.
After the response from the remote server is received, the proxy should send the response message (as-is) to the client via the appropriate socket. Once the transaction is complete, the proxy should close the connection to the client. Note: the proxy should terminate the connection to the remote server once the response has been fully received. For HTTP 1.0, the remote server will terminate the connection once the transaction is complete.
> Caching 
Just before you send back the response to the client, you make a copy and store this response somewhere in memory (like a hashtable) not in a file.
If the client sends a request twice, you’ll use the cached copy the second time and later on (Never mind cache expiry, we’ll assume that the page is totally static). Caching is explained in the Proxy video in resources section.
Implement caching to your proxy server. To test this feature, make a request twice and you must notice a greatly reduced response time.
> Serve the masses
By ending this part, you should be able to serve clients in parallel, either using threads, nonblocking sockets or using asyncio (advanced), This part will add a good amount of complexity to your code as you’ll need to add more things to make sure each client is handled correctly.
Make sure you have a backup of the single-threaded proxy. You can submit it if things go wrong. Using git is also a super good idea, please consider it.
To serve multiple clients at once you can select one of the following methods, sorted by difficulty from low to high
   * Threading (a thread per client)
   * select() / non-blocking sockets
   * Implementing a thread pool and queuing jobs into it
   * Using asynchronous programming
# Logic Flow
Your proxy server is expected to do the following (assuming you’re single threaded)
   1. Open a listening TCP socket, and binds it to the port specified by the first command line argument
   2. Use TCP for communication with both clients and servers.
   3. Open a TCP connection for every HTTP request to be proxied. 
   4. After the client’s HTTP request is parsed. Open a TCP connection with the asked website, making the request and return the response back to the client if successful.
   5. Close the TCP connection with both client and requested website’s server. (because HTTP 1.0 doesn’t have persistent connections)
   
   
# Testing your proxy 

Open a terminal, then run your proxy (assuming that you’re using Python), your proxy will run on 127.0.0.1 as always.
~ python proxy.py 2233
	Now, leave your proxy running then issue the telnet command (we’re telnetting to the proxy now -not a website-)
~ telnet 127.0.0.1 2233
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
GET http://info.cern.ch:80/ HTTP/1.0
- hit enter twice
	Now you should get a reply similar to the first one.
Note if you want to attach an HTTP header to the telnet command, hit enter once then enter the header you want.
~ telnet info.cern.ch 80
Trying 188.184.64.53...
Connected to info.cern.ch.
Escape character is '^]'.
GET /hypertext/WWW/TheProject.html HTTP/1.0
Host: info.cern.ch # hit enter twice
	You can also use curl (proxy config) [make sure you pass --http1.0 flag] 
curl --http1.0 --proxy 127.0.0.1:8080 -X GET http://www.apache.org
	keep testing with the browser (search how to use it -prefer firefox-) the last thing you do. Since it sends lots of stuff at once. Postman is a nice GUI tool that you can use to test your proxy. You can find it here, it’s always used for testing HTTP APIs in companies, you can check out the first 3 videos of this playlist to know how to use it. You’ll be doing your future-self a favor if you learn it now. First, you’ll configure postman to use your server as a proxy [guide] then send your requests normally.
Using Telnet to verify your output
A good sanity check of proxy behavior would be to compare the HTTP response (headers and body) obtained via your proxy by calling telnet to your proxy, then making an HTTP request from the command line with the response from a direct telnet connection to the remote server. Check that by capturing the HTTP GET packet and compare the result.
~ telnet info.cern.ch 80
Trying 188.184.64.53...
Connected to info.cern.ch.
Escape character is '^]'.
GET /hypertext/WWW/TheProject.html HTTP/1.0
Host: info.cern.ch # hit enter twice
