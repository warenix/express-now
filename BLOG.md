# How to Deploy Express on Now.sh

In this post I'm going to share how to setup express API endpoints to run on version 2 of [Now.sh][now_v2]. You will get a free https endpoints and run in serverless! Isn't it cool?

You can find full source code at [github][github_repo].

## Prerequisite

-   Now CLI (12.1.9)
-   Node (v10.10.0)
-   express (4.16.4)

## Add Endpoints to express

For simplicity we are going to have 2 endpoints to show how to handle `GET` and `POST` requests.

## /get - GET

This returns `VERSION` in json output.

Edit `index.js`

```js
app.get("/get", (req, res, next) => {
    res.json({
        "version": process.env.VERSION
    });
});
```

## /post - POST

Echo back JSON content being posted.

Edit `index.js`

```js
app.post('/post', function(request, response) {
    response.send(request.body);
});
```

## Storing Secret as Environment Variable

You may have noticed in the '/get' endpoint we used `process.env.VERSION`. This is a common practice not to hardcode secrets in code.

### Set Environment Variables

```sh
export VERSION="1.0"
```

## Deploy to `now.sh`

### Setup Build for `now`

We need to setup `build` to use `@now/node-server`. (Using `@now/node` just won't work). Modify `now.json`

```json
"builds": [{
    "src": "index.js",
    "use": "@now/node-server"
}]
```

Read more at [doc][doc_build].

### Set Environment Variable as Secret in `now.sh`

```sh
now-linux secret add VERSION $VERSION
```

Read more at [doc][doc_secret].

### Allow CORS

Here we need to add custom response headers. Modify `now.json`

```json
"routes": [{
    "headers": {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
        "Access-Control-Allow-Headers": "X-Requested-With, Content-Type, Accept"
    },
    "src": "/.*",
    "dest": "/index.js"
}]
```

Read more at [doc][doc_headers].

### Push to `now.sh`

```sh
now-linux
```

Sample output

```sh
❯ now-linux
> UPDATE AVAILABLE The latest version of Now CLI is 12.1.9
> Read more about how to update here: https://zeit.co/update-cli
> Changelog: https://github.com/zeit/now-cli/releases/tag/12.1.9
> Deploying ~/code/repo/github/express-now under XXXXXXX
> Synced 2 files (929B) [1s]
> https://express-now-3b57ke4d4.now.sh [v2] [in clipboard] [1s]
┌ index.js        Ready               [17s]
└── λ index.js (284.31KB) [sfo1]
> Success! Deployment ready [19s]
```

## Tests

Spin up a localhost server.

```sh
npm start
```

## Test `/get`

In terminal,

```sh
curl http://localhost:3000/get
```

Response

```sh
{"version":"1.0"}
```

## Test `/post`

In terminal,

```sh
curl -H "Content-Type: application/json" \
-d '{"message":"hello"}' \
http://localhost:3000/post
```

Response

```sh
{"message":"hello"}
```

Note: You can replace `localhost` with the now.sh instance url.

## Gotcha

Perhaps due to the nature of serverless sometime the endpoint returns `502` error. To tackle that please add retry mechanism to your service callers.

[github_repo]: https://github.com/warenix/express-now

[now_v2]: https://zeit.co/docs/v2/getting-started/introduction-to-now

[doc_build]: https://zeit.co/docs/v2/deployments/official-builders/node-js-server-now-node-server/

[doc_secret]: https://zeit.co/docs/v2/deployments/environment-variables-and-secrets/

[doc_headers]: https://zeit.co/docs/v2/deployments/configuration/#routes
