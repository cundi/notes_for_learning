There's two way you can work around it

Make sure your request is a "simple request"
Set Access-Control-Max-Age for the OPTION request


Have gone through this issue, below is my conclusion to this issue and my solution.

According to the CORS strategy (highly recommend you read about it) You can't just force the browser to stop sending OPTION request if it thinks it needs to.

There's two way you can work around it

Make sure your request is a "simple request"
Set Access-Control-Max-Age for the OPTION request
Simple request
A simple cross-site request is one that meets all the following conditions:

The only allowed methods are: - GET - HEAD - POST

Apart from the headers set automatically by the user agent (e.g. Connection, User-Agent, etc.), the only headers which are allowed to be manually set are: - Accept - Accept-Language - Content-Language - Content-Type

The only allowed values for the Content-Type header are: - application/x-www-form-urlencoded - multipart/form-data - text/plain

A simple request will not cause a pre-flight OPTION request.

Set a cache for the OPTION check
You can set a Access-Control-Max-Age for the OPTION request, so that it will not check the permission again until it is expired.

Access-Control-Max-Age gives the value in seconds for how long the response to the preflight request can be cached for without sending another preflight request.