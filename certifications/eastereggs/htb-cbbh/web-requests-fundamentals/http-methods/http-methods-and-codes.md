# HTTP Methods & Codes

## Request Methods

| **Method** | **Description**                                                                                                                                                                                                                                                                                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `GET`      | Requests a specific resource. Additional data can be passed to the server via query strings in the URL (e.g. `?param=value`).                                                                                                                                                                                                        |
| `POST`     | Sends data to the server. It can handle multiple types of input, such as text, PDFs, and other forms of binary data. This data is appended in the request body present after the headers. The POST method is commonly used when sending information (e.g. forms/logins) or uploading data to a website, such as images or documents. |
| `HEAD`     | Requests the headers that would be returned if a GET request was made to the server. It doesn't return the request body and is usually made to check the response length before downloading resources.                                                                                                                               |
| `PUT`      | Creates new resources on the server. Allowing this method without proper controls can lead to uploading malicious resources.                                                                                                                                                                                                         |
| `DELETE`   | Deletes an existing resource on the webserver. If not properly secured, can lead to Denial of Service (DoS) by deleting critical files on the web server.                                                                                                                                                                            |
| `OPTIONS`  | Returns information about the server, such as the methods accepted by it.                                                                                                                                                                                                                                                            |
| `PATCH`    | Applies partial modifications to the resource at the specified location.                                                                                                                                                                                                                                                             |

## Response Codes

| **Code**                    | **Description**                                                                                                                                           |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `200 OK`                    | Returned on a successful request, and the response body usually contains the requested resource.                                                          |
| `302 Found`                 | Redirects the client to another URL. For example, redirecting the user to their dashboard after a successful login.                                       |
| `400 Bad Request`           | Returned on encountering malformed requests such as requests with missing line terminators.                                                               |
| `403 Forbidden`             | Signifies that the client doesn't have appropriate access to the resource. It can also be returned when the server detects malicious input from the user. |
| `404 Not Found`             | Returned when the client requests a resource that doesn't exist on the server.                                                                            |
| `500 Internal Server Error` | Returned when the server cannot process the request.                                                                                                      |
