---
layout: "../../layouts/BlogPostLayout.astro"
title: REST API & XHR,Axios,Fetch
date: 2023-04-03
author: Teki
image: {
  src: "/images/rest/cover.png",
  alt: "cover image",
}
description: Basic concept of REST API and some usage examples at frontend side.
draft: false
category: Coding
---
### Why we need REST API?

#### structure of traditional server: MVC---Model View Controller

- Model: data model, interact with database
- View: a view for rendering (ex. a HTML page with template)
- Controller: Main logic of App(ex. CRUD), using data model to load and process data and render data to certain view page

#### problems of MVC structure

- When a App has several clients(such as web App, mobile App), several servers will be needed.
- Not good for uncoupling of frontend and backend.

#### How to solve them --- REST API

### Rest API

- REpresentational State Transfer Application Progrmming Interface
- a style for system design
- main property: moving data render and view to frontend, server only takes charge of returning data
- data format: JSON
- Request methods:
  - GET: fetch data
  - POST: new/add data
  - PUT: add/modify data
  - PATCH: modify data
  - DELETE: delete data
  - OPTION: automatically sent by browser for some check

- Example:
  - GET /user
  - POST /user
  - DELETE /user/:id

### XHR(XML HTTP Request)

#### 1. Examples

```javascript
// GET
// init a new XHR instance
const xhr = new XMLHttpRequest()
// open a request
xhr.open("GET", "http://localhost:3000/students")
// send request
xhr.send()
```

```javascript
// POST
const xhr = new XMLHttpRequest();
xhr.open("POST", '/server', true);

//Send the proper header information along with the request
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

xhr.onreadystatechange = () => { // Call a function when the state changes.
  if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
    // Request finished. Do processing here.
  }
}
xhr.send("foo=bar&lorem=ipsum");
```

#### 2. Instance Methods

- xhr.open():

```javascript
open(method, url, async, user, password)
```

parameters:
  / **method**:The HTTP request method to use, such as "GET", "POST", "PUT", "DELETE", etc. Ignored for non-HTTP(S) URLs.
  / **url**: A string representing the URL to send the request to.
  / **async**(optional): An optional Boolean parameter, defaulting to true, indicating whether or not to perform the operation asynchronously. If this value is false, the send() method does not return until the response is received. If true, notification of a completed transaction is provided using event listeners. This must be true if the multipart attribute is true, or an exception will be thrown.
  / **user**(optional): The optional user name to use for authentication purposes; by default, this is the null value.
  / **password**: The optional password to use for authentication purposes; by default, this is the null value.

- xhr.send():

```javascript
send(body)
```

parameters:

  / **body**(optional):
A body of data to be sent in the XHR request. This can be:

- A Document, in which case it is serialized before being sent.
- An XMLHttpRequestBodyInit, which per the Fetch spec can be a Blob, an ArrayBuffer, a TypedArray, a DataView, a FormData, a URLSearchParams, or a string literal or object.
- null

If no value is specified for the body, a default value of null is used.

- xhr.abort()
- xhr.setRequestHeader()

```javascript
setRequestHeader(header, value)
```

- xhr.getResponseHeader()

```javascript
getResponseHeader(headerName)
```

- xhr.getAllResponseHeaders()

```javascript
getAllResponseHeaders()
```

- xhr.overrideMimeType()

```javascript
overrideMimeType(mimeType)
```

example:

```javascript
// Interpret the received data as plain text

req = new XMLHttpRequest();
req.overrideMimeType("text/plain");
req.addEventListener("load", callback, false);
req.open("get", url);
req.send();
```

#### 3.Events

- abort

```javascript
addEventListener('abort', (event) => { })

onabort = (event) => { }
```

- error

```javascript
addEventListener('error', (event) => { })

onerror = (event) => { }
```

- load

```javascript
addEventListener('load', (event) => { })

onload = (event) => { }
```

- loadend

```javascript
addEventListener('loadend', (event) => { })

onloadend = (event) => { }
```

- loadstart

```javascript
addEventListener('loadstart', (event) => { })

onloadstart = (event) => { }
```

- progress

```javascript
addEventListener('progress', (event) => { })

onprogress = (event) => { }
```

- readystatechange

```javascript
addEventListener('readystatechange', (event) => { })

onreadystatechange = (event) => { }
```

Example:

```javascript
const xhr = new XMLHttpRequest();
const method = "GET";
const url = "https://developer.mozilla.org/";

xhr.open(method, url, true);
xhr.onreadystatechange = () => {
  // In local files, status is 0 upon success in Mozilla Firefox
  if (xhr.readyState === XMLHttpRequest.DONE) {
    const status = xhr.status;
    if (status === 0 || (status >= 200 && status < 400)) {
      // The request has been completed successfully
      console.log(xhr.responseText);
    } else {
      // Oh no! There has been an error with the request!
    }
  }
};
xhr.send();
```

- timeout

```javascript
addEventListener('timeout', (event) => { })

ontimeout = (event) => { }
```

Example:

```javascript
const client = new XMLHttpRequest();
client.open('GET', 'http://www.example.org/example.txt');
client.ontimeout = () => {
    console.error('Timeout!!')
};

client.send();
```

### Fetch

#### 1. Examples

```javascript
// GET
async function logJSONData() {
  const response = await fetch("http://example.com/movies.json");
  const jsonData = await response.json();
  console.log(data);
}
```

```javascript
// POST
async function postData(url = "", data = {}) {
  // Default options are marked with *
  const response = await fetch(url, {
    method: "POST", // *GET, POST, PUT, DELETE, etc.
    mode: "cors", // no-cors, *cors, same-origin
    cache: "no-cache", // *default, no-cache, reload, force-cache, only-if-cached
    credentials: "same-origin", // include, *same-origin, omit
    headers: {
      "Content-Type": "application/json",
      // 'Content-Type': 'application/x-www-form-urlencoded',
    },
    redirect: "follow", // manual, *follow, error
    referrerPolicy: "no-referrer", // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
    body: JSON.stringify(data), // body data type must match "Content-Type" header
  });
  return response.json(); // parses JSON response into native JavaScript objects
}

postData("https://example.com/answer", { answer: 42 }).then((data) => {
  console.log(data); // JSON data parsed by `data.json()` call
});
```

```javascript
// Using controller to abort a Fetch
const controller = new AbortController();
const signal = controller.signal;
const url = "video.mp4";

const downloadBtn = document.querySelector("#download");
const abortBtn = document.querySelector("#abort");

downloadBtn.addEventListener("click", async () => {
  try {
    const response = await fetch(url, { signal });
    console.log("Download complete", response);
  } catch (error) {
    console.error(`Download error: ${error.message}`);
  }
});

abortBtn.addEventListener("click", () => {
  controller.abort();
  console.log("Download aborted");
});
```

```javascript
// Uploading JSON data
async function postJSON(data) {
  try {
    const response = await fetch("https://example.com/profile", {
      method: "POST", // or 'PUT'
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data),
    });

    const result = await response.json();
    console.log("Success:", result);
  } catch (error) {
    console.error("Error:", error);
  }
}

const data = { username: "example" };
postJSON(data);
```

```javascript
// Uploading a file
async function upload(formData) {
  try {
    const response = await fetch("https://example.com/profile/avatar", {
      method: "PUT",
      body: formData,
    });
    const result = await response.json();
    console.log("Success:", result);
  } catch (error) {
    console.error("Error:", error);
  }
}

const formData = new FormData();
const fileField = document.querySelector('input[type="file"]');

formData.append("username", "abc123");
formData.append("avatar", fileField.files[0]);

upload(formData);
```

```javascript
// Checking that the Fetch was successful
async function fetchImage() {
  try {
    const response = await fetch("flowers.jpg");
    if (!response.ok) {
      throw new Error("Network response was not OK");
    }
    const myBlob = await response.blob();
    myImage.src = URL.createObjectURL(myBlob);
  } catch (error) {
    console.error("There has been a problem with your fetch operation:", error);
  }
}
```

#### 2. Headers

- instance methods:
  - Headers.append()
  - Headers.delete()
  - Headers.entries()
  - Headers.forEach()
  - Headers.get()
  - Headers.has()
  - Headers.keys()
  - Headers.set()
  - Headers.values()
Example:

```javascript
const myHeaders = new Headers();

myHeaders.append("Content-Type", "text/xml");
myHeaders.get("Content-Type"); // should return 'text/xml'
```

#### 3. Request

Example:

```javascript
const request = new Request("https://example.com", {
  method: "POST",
  body: '{"foo": "bar"}',
});

const url = request.url;
const method = request.method;
const credentials = request.credentials;
const bodyUsed = request.bodyUsed;
```

- Instance Properties
  - **Request.body Read only**
A ReadableStream of the body contents.

  - **Request.bodyUsed Read only**
Stores true or false to indicate whether or not the body has been used in a request yet.

  - **Request.cache Read only**
Contains the cache mode of the request (e.g., default, reload, no-cache).

  - **Request.credentials Read only**
Contains the credentials of the request (e.g., omit, same-origin, include). The default is same-origin.

  - **Request.destination Read only**
Returns a string describing the request's destination. This is a string indicating the type of content being requested.

  - **Request.headers Read only**
Contains the associated Headers object of the request.

  - **Request.integrity Read only**
Contains the subresource integrity value of the request (e.g., sha256-BpfBw7ivV8q2jLiT13fxDYAe2tJllusRSZ273h2nFSE=).

  - **Request.method Read only**
Contains the request's method (GET, POST, etc.)

  - **Request.mode Read only**
Contains the mode of the request (e.g., cors, no-cors, same-origin, navigate.)

  - **Request.redirect Read only**
Contains the mode for how redirects are handled. It may be one of follow, error, or manual.

  - **Request.referrer Read only**
Contains the referrer of the request (e.g., client).

  - **Request.referrerPolicy Read only**
Contains the referrer policy of the request (e.g., no-referrer).

  - **Request.signal Read only**
Returns the AbortSignal associated with the request

  - **Request.url Read only**
Contains the URL of the request.

- Instance Methods
  - **Request.arrayBuffer()**
Returns a promise that resolves with an ArrayBuffer representation of the request body.

  - **Request.blob()**
Returns a promise that resolves with a Blob representation of the request body.

  - **Request.clone()**
Creates a copy of the current Request object.

  - **Request.formData()**
Returns a promise that resolves with a FormData representation of the request body.

  - **Request.json()**
Returns a promise that resolves with the result of parsing the request body as JSON.

  - **Request.text()**
Returns a promise that resolves with a text representation of the request body.

#### 4. Response

- Example:

```javascript
const image = document.querySelector(".my-image");
fetch("flowers.jpg")
  .then((response) => response.blob())
  .then((blob) => {
    const objectURL = URL.createObjectURL(blob);
    image.src = objectURL;
  });
```

- Instance Properties
  - **Response.body** Read only
A ReadableStream of the body contents.

  - **Response.bodyUsed** Read only
Stores a boolean value that declares whether the body has been used in a response yet.

  - **Response.headers** Read only
The Headers object associated with the response.

  - **Response.ok** Read only
A boolean indicating whether the response was successful (status in the range 200 – 299) or not.

  - **Response.redirected** Read only
Indicates whether or not the response is the result of a redirect (that is, its URL list has more than one entry).

  - **Response.status** Read only
The status code of the response. (This will be 200 for a success).

  - **Response.statusText** Read only
The status message corresponding to the status code. (e.g., OK for 200).

  - **Response.type** Read only
The type of the response (e.g., basic, cors).

  - **url** Read only
The URL of the response.

- Static methods
  - **Response.error()**
Returns a new Response object associated with a network error.

  - **Response.redirect()**
Creates a new response with a different URL.

- Instance methods

  - **Response.arrayBuffer()**
Returns a promise that resolves with an ArrayBuffer representation of the response body.

  - **Response.blob()**
Returns a promise that resolves with a Blob representation of the response body.

  - **Response.clone()**
Creates a clone of a Response object.

  - **Response.formData()**
Returns a promise that resolves with a FormData representation of the response body.

  - **Response.json()**
Returns a promise that resolves with the result of parsing the response body text as JSON.

  - **Response.text()**
Returns a promise that resolves with a text representation of the response body.

### Axios

#### 1. What is Axios

Axios is a promise-based HTTP Client for node.js and the browser. It is isomorphic (= it can run in the browser and nodejs with the same codebase). On the server-side it uses the native node.js http module, while on the client (browser) it uses XMLHttpRequests.

#### 2. Example

```javascript
// GET
const axios = require('axios');

// Make a request for a user with a given ID
axios.get('/user?ID=12345')
  .then(function (response) {
    // handle success
    console.log(response);
  })
  .catch(function (error) {
    // handle error
    console.log(error);
  })
  .finally(function () {
    // always executed
  });

// Optionally the request above could also be done as
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .finally(function () {
    // always executed
  });  

// Want to use async/await? Add the `async` keyword to your outer function/method.
async function getUser() {
  try {
    const response = await axios.get('/user?ID=12345');
    console.log(response);
  } catch (error) {
    console.error(error);
  }
}
```

```javascript
// POST
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

```javascript
// Multiple concurrent requests
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

const [acct, perm] = await Promise.all([getUserAccount(), getUserPermissions()]);

// OR

Promise.all([getUserAccount(), getUserPermissions()])
  .then(function ([acct, perm]) {
    // ...
  });
```

```javascript
// Post an HTML form as JSON
const {data} = await axios.post('/user', document.querySelector('#my-form'), {
  headers: {
    'Content-Type': 'application/json'
  }
})
```

#### 3. Axios API

- axios(config)

```javascript
// Send a POST request
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```

```javascript
// GET request for remote image in node.js
axios({
  method: 'get',
  url: 'http://bit.ly/2mTM3nY',
  responseType: 'stream'
})
  .then(function (response) {
    response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
  });
```

#### 4. Axios instance

axios.create([config])

```javascript
const instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

#### 5. Request Config

These are the available config options for making requests. Only the url
 is required. Requests will default to GET
 if method is not specified.

```javascript
{
  // `url` is the server URL that will be used for the request
  url: '/user',

  // `method` is the request method to be used when making the request
  method: 'get', // default

  // `baseURL` will be prepended to `url` unless `url` is absolute.
  // It can be convenient to set `baseURL` for an instance of axios to pass relative URLs
  // to methods of that instance.
  baseURL: 'https://some-domain.com/api',

  // `transformRequest` allows changes to the request data before it is sent to the server
  // This is only applicable for request methods 'PUT', 'POST', 'PATCH' and 'DELETE'
  // The last function in the array must return a string or an instance of Buffer, ArrayBuffer,
  // FormData or Stream
  // You may modify the headers object.
  transformRequest: [function (data, headers) {
    // Do whatever you want to transform the data

    return data;
  }],

  // `transformResponse` allows changes to the response data to be made before
  // it is passed to then/catch
  transformResponse: [function (data) {
    // Do whatever you want to transform the data

    return data;
  }],

  // `headers` are custom headers to be sent
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` are the URL parameters to be sent with the request
  // Must be a plain object or a URLSearchParams object
  // NOTE: params that are null or undefined are not rendered in the URL.
  params: {
    ID: 12345
  },

  // `paramsSerializer` is an optional function in charge of serializing `params`
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function (params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` is the data to be sent as the request body
  // Only applicable for request methods 'PUT', 'POST', 'DELETE', and 'PATCH'
  // When no `transformRequest` is set, must be of one of the following types:
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - Browser only: FormData, File, Blob
  // - Node only: Stream, Buffer
  data: {
    firstName: 'Fred'
  },
  
  // syntax alternative to send data into the body
  // method post
  // only the value is sent, not the key
  data: 'Country=Brasil&City=Belo Horizonte',

  // `timeout` specifies the number of milliseconds before the request times out.
  // If the request takes longer than `timeout`, the request will be aborted.
  timeout: 1000, // default is `0` (no timeout)

  // `withCredentials` indicates whether or not cross-site Access-Control requests
  // should be made using credentials
  withCredentials: false, // default

  // `adapter` allows custom handling of requests which makes testing easier.
  // Return a promise and supply a valid response (see lib/adapters/README.md).
  adapter: function (config) {
    /* ... */
  },

  // `auth` indicates that HTTP Basic auth should be used, and supplies credentials.
  // This will set an `Authorization` header, overwriting any existing
  // `Authorization` custom headers you have set using `headers`.
  // Please note that only HTTP Basic auth is configurable through this parameter.
  // For Bearer tokens and such, use `Authorization` custom headers instead.
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` indicates the type of data that the server will respond with
  // options are: 'arraybuffer', 'document', 'json', 'text', 'stream'
  //   browser only: 'blob'
  responseType: 'json', // default

  // `responseEncoding` indicates encoding to use for decoding responses (Node.js only)
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // default

  // `xsrfCookieName` is the name of the cookie to use as a value for xsrf token
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
  xsrfHeaderName: 'X-XSRF-TOKEN', // default

  // `onUploadProgress` allows handling of progress events for uploads
  // browser only
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `onDownloadProgress` allows handling of progress events for downloads
  // browser only
  onDownloadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `maxContentLength` defines the max size of the http response content in bytes allowed in node.js
  maxContentLength: 2000,

  // `maxBodyLength` (Node only option) defines the max size of the http request content in bytes allowed
  maxBodyLength: 2000,

  // `validateStatus` defines whether to resolve or reject the promise for a given
  // HTTP response status code. If `validateStatus` returns `true` (or is set to `null`
  // or `undefined`), the promise will be resolved; otherwise, the promise will be
  // rejected.
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },

  // `maxRedirects` defines the maximum number of redirects to follow in node.js.
  // If set to 0, no redirects will be followed.
  maxRedirects: 5, // default

  // `socketPath` defines a UNIX Socket to be used in node.js.
  // e.g. '/var/run/docker.sock' to send requests to the docker daemon.
  // Only either `socketPath` or `proxy` can be specified.
  // If both are specified, `socketPath` is used.
  socketPath: null, // default

  // `httpAgent` and `httpsAgent` define a custom agent to be used when performing http
  // and https requests, respectively, in node.js. This allows options to be added like
  // `keepAlive` that are not enabled by default.
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // `proxy` defines the hostname, port, and protocol of the proxy server.
  // You can also define your proxy using the conventional `http_proxy` and
  // `https_proxy` environment variables. If you are using environment variables
  // for your proxy configuration, you can also define a `no_proxy` environment
  // variable as a comma-separated list of domains that should not be proxied.
  // Use `false` to disable proxies, ignoring environment variables.
  // `auth` indicates that HTTP Basic auth should be used to connect to the proxy, and
  // supplies credentials.
  // This will set an `Proxy-Authorization` header, overwriting any existing
  // `Proxy-Authorization` custom headers you have set using `headers`.
  // If the proxy server uses HTTPS, then you must set the protocol to `https`. 
  proxy: {
    protocol: 'https',
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` specifies a cancel token that can be used to cancel the request
  // (see Cancellation section below for details)
  cancelToken: new CancelToken(function (cancel) {
  }),

  // `decompress` indicates whether or not the response body should be decompressed 
  // automatically. If set to `true` will also remove the 'content-encoding' header 
  // from the responses objects of all decompressed responses
  // - Node only (XHR cannot turn off decompression)
  decompress: true // default

}
```

#### 6. Response Schema

```javascript
{
  // `data` is the response that was provided by the server
  data: {},

  // `status` is the HTTP status code from the server response
  status: 200,

  // `statusText` is the HTTP status message from the server response
  // As of HTTP/2 status text is blank or unsupported.
  // (HTTP/2 RFC: https://www.rfc-editor.org/rfc/rfc7540#section-8.1.2.4)
  statusText: 'OK',

  // `headers` the HTTP headers that the server responded with
  // All header names are lower cased and can be accessed using the bracket notation.
  // Example: `response.headers['content-type']`
  headers: {},

  // `config` is the config that was provided to `axios` for the request
  config: {},

  // `request` is the request that generated this response
  // It is the last ClientRequest instance in node.js (in redirects)
  // and an XMLHttpRequest instance in the browser
  request: {}
}
```

```javascript
axios.get('/user/12345')
  .then(function (response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });
```

#### 7. Config defaults

```javascript
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

#### 8. Interceptors

You can intercept requests or responses before they are handled by then or catch.

```javascript
// Add a request interceptor
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  }, function (error) {
    // Do something with request error
    return Promise.reject(error);
  });

// Add a response interceptor
axios.interceptors.response.use(function (response) {
    // Any status code that lie within the range of 2xx cause this function to trigger
    // Do something with response data
    return response;
  }, function (error) {
    // Any status codes that falls outside the range of 2xx cause this function to trigger
    // Do something with response error
    return Promise.reject(error);
  });
  ```

  ```javascript
  // Remove an interceptor
  const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

#### 9. Handling Errors

```javascript
axios.get('/user/12345')
  .catch(function (error) {
    if (error.response) {
      // The request was made and the server responded with a status code
      // that falls out of the range of 2xx
      console.log(error.response.data);
      console.log(error.response.status);
      console.log(error.response.headers);
    } else if (error.request) {
      // The request was made but no response was received
      // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
      // http.ClientRequest in node.js
      console.log(error.request);
    } else {
      // Something happened in setting up the request that triggered an Error
      console.log('Error', error.message);
    }
    console.log(error.config);
  });
  ```

  ```javascript
  axios.get('/user/12345', {
  validateStatus: function (status) {
    return status < 500; // Resolve only if the status code is less than 500
  }
})
```

```javascript
axios.get('/user/12345')
  .catch(function (error) {
    console.log(error.toJSON());
  });
```

#### 10. Cancellation

```javascript
const controller = new AbortController();

axios.get('/foo/bar', {
   signal: controller.signal
}).then(function(response) {
   //...
});
// cancel the request
controller.abort()
```

```javascript
axios.get('/foo/bar', {
   signal: AbortSignal.timeout(5000) //Aborts request after 5 seconds
}).then(function(response) {
   //...
});
```

```javascript
function newAbortSignal(timeoutMs) {
  const abortController = new AbortController();
  setTimeout(() => abortController.abort(), timeoutMs || 0);

  return abortController.signal;
}

axios.get('/foo/bar', {
   signal: newAbortSignal(5000) //Aborts request after 5 seconds
}).then(function(response) {
   //...
});
```

#### 11. URL-Encoding Bodies

By default, axios serializes JavaScript objects to JSON. To send data in the application/x-www-form-urlencodedformat instead, you can use one of the following approaches.

```javascript
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);
```

```javascript
const qs = require('qs');
axios.post('/foo', qs.stringify({ 'bar': 123 }));
//or
import qs from 'qs';
const data = { 'bar': 123 };
const options = {
  method: 'POST',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  data: qs.stringify(data),
  url,
};
axios(options);
```

#### 12. Multipart Bodies

Posting data as multipart/form-data.

```javascript
const form = new FormData();
form.append('my_field', 'my value');
form.append('my_buffer', new Blob([1,2,3]));
form.append('my_file', fileInput.files[0]);

axios.post('https://example.com', form)

//or 

axios.postForm('https://httpbin.org/post', {
  my_field: 'my value',
  my_buffer: new Blob([1,2,3]),
  my_file:  fileInput.files // FileList will be unwrapped as sepate fields
});
```

Starting from v0.27.0, Axios supports automatic object serialization to a FormData object if the request Content-Type header is set to multipart/form-data.

```javascript
import axios from 'axios';

axios.post('https://httpbin.org/post', {
  user: {
    name: 'Dmitriy'
  },
  file: fs.createReadStream('/foo/bar.jpg')
}, {
  headers: {
    'Content-Type': 'multipart/form-data'
  }
}).then(({data})=> console.log(data));
```

#### 13. An example of Axios Encapsulation

utils/http.js

```javascript
import axios from 'axios'
import { clearToken, getToken } from './token'
import { history } from './history'
const http = axios.create({
  baseURL: 'http://geek.itheima.net/v1_0',
  timeout:5000
})

http.interceptors.request.use((config)=>{
  const token = getToken()
  if(token){
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
},(error)=>{
  return Promise.reject(error)
})

http.interceptors.response.use((response)=>{
  return response
},(error)=>{
  if(error.response.status===401){
    clearToken()
    history.push('/login')
  }
  return Promise.reject(error)
})

export {http}
```

utils/token.js

```javascript
const TOKEN_KEY = 'geek_pc'

const getToken = ()=>localStorage.getItem(TOKEN_KEY)
const setToken = (token)=>localStorage.setItem(TOKEN_KEY,token)
const clearToken = ()=>localStorage.removeItem(TOKEN_KEY)

export {getToken,setToken,clearToken}
```

utils/index.js

```javascript
import {http} from './http'
import { getToken,setToken,clearToken } from './token'

export {http,getToken,setToken,clearToken}
```

#### 14. An example of Route Guard

Components/AuthRoute.js

```javascript
import { getToken } from "../../utils/token";
import {Navigate} from 'react-router-dom'

function AuthRoute ({children}) {
  const isToken = getToken()
  if(isToken){
    return <>{children}</>
  }else{
    return <Navigate to="/login" replace></Navigate>
  }
}
export {AuthRoute}
```

App.js

```javascript
import { HistoryRouter,history } from './utils/history';
import {BrowserRouter,Route,Routes} from 'react-router-dom'
import Login from './pages/Login';
import Home from './pages/Home';
import Article from './pages/Article';
import Publish from './pages/Publish';
import Layout from './pages/Layout';
import { AuthRoute } from './components/AuthRoute';
function App() {
  return (
  <HistoryRouter history={history}>
      <div className='App'>
        <Routes>
          <Route path='/*' element={
            <AuthRoute>
              <Layout />
            </AuthRoute>}> 
            <Route index element={<Home />} />
            <Route path='article' element={<Article />} />
            <Route path='publish' element={<Publish />} />
          </Route>
          <Route path='/login' element={<Login />} />
        </Routes>
      </div>
  </HistoryRouter>
  );
}

export default App;
```

### Cross origin

Solve cross-origin problem at server side by setting the response header.

```javascript
app.use((req, res, next) => {
    // 设置响应头
    res.setHeader("Access-Control-Allow-Origin", "*")
    res.setHeader("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,PATCH")
    res.setHeader("Access-Control-Allow-Headers", "Content-type")
    next()
})
```
