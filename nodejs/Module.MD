--> nodejs module

NodeJS
    |--module
        |--router.js
        |--server.js
    index.js
-------------------------------------------------------------
index.js：
    var server=require("./module/server");
    var router=require("./module/router");
    server.start(router.route);
-------------------------------------------------------------
server.js:
    var http = require("http");
    var url=require("url");
    
    function start(route) {
        function onRequest(request, response) {
            var pathname=url.parse(request.url).pathname;
            route(pathname);
            response.writeHead(200, {
                "Content-Type": "text/plain"
            });
            response.write("hello,world");
            response.end();
        }
        http.createServer(onRequest).listen(8080);
    }
    exports.start=start;
-------------------------------------------------------------
router.js：
    function route(pathname){
        console.log("message from"+pathname+"...");
    }
    exports.route=route;