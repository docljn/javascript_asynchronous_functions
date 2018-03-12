# Coming soon!

## Promisifying AJAX
### XMLHttpRequest

#### Original code: using a callback (AJAX)

1. make an instance of an object that can make an HTTP request
```javascript
httpRequest = new XMLHttpRequest();
```
2. tell the XMLHttp request object which JavaScript function will handle the request response
```javascript
httpRequest.onreadystatechange = function () {
  // process server response here
};
```
3. make the request
  - the third argument defaults to true i.e. asynchronous
```javascript
httpRequest.open("POST", "url_to_send_request_to", true);
httpRequest.setRequestHeader("Content-Type", "application/json");
httpRequest.send("data_to_send_to_server");
```
4. Check the state of the request: if it's DONE then continue
```javascript
if (httpRequest.readyState === XMLHttpRequest.DONE) {
  // Everything is good: response received
} else {
  // Not ready yet
}
```
5. Check the HTTP response code i.e. the server call status: 200 is OK
```javascript
if (httpRequest.status = 200) {
  // Perfect
} else {
  // There was a problem with the request
  // eg. 404 not found, 500 internal server error
};
```
6. Actually use the data that the server sent
```javascript
const serverResponse = httpRequest.responseText
```

#### Actually making an XMLHttpRequest
```javascript
function() {
  var httpRequest;
  document.getElementById("ajaxButton").addEventListener('click', makeRequest);

  function makeRequest() {
    httpRequest = new XMLHttpRequest();

    if (!httpRequest) {
      alert('Giving up :( Cannot create an XMLHTTP instance');
      return false;
    }
    httpRequest.onreadystatechange = alertContents;
    httpRequest.open('GET', 'test.html');
    httpRequest.send();
  }

  function alertContents() {
    try {
      if (httpRequest.readyState === XMLHttpRequest.DONE) {
        if (httpRequest.status === 200) {
          alert(httpRequest.responseText);
        } else {
          alert('There was a problem with the request.');
        }
      }
    }
    catch( e ) {
      alert('Caught Exception: ' + e.description);
    }
  }
}
```

So, to actually make an XMLHttpRequest, our notes do:
