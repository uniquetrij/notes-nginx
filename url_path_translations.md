
# Understanding Nginx proxy_pass and url path translations

Consider the following nginx config blocks

location / {
    proxy_pass http://127.0.0.1:8080;
}

and

location / {
    proxy_pass http://127.0.0.1:8080/;
}

They are both same in terms of final result. You if call /abc/def on the main server it passes the request to http://127.0.0.1:8080/abc/def

But as soon as you have a location block with a non-root path like below

location /app {
    proxy_pass http://127.0.0.1:8080;
}

The above config is not same as below

location /app/ {
    proxy_pass http://127.0.0.1:8080/;
}

Below table shows different combinations of the location and the proxy_pass url and the final path that the backend server will receive
Proxy Pass and Paths
Case # 	Nginx location 	proxy_pass URL 	Test URL 	Path received
1 	/test1 	http://127.0.0.1:8080 	/test1/abc/test 	/test1/abc/test
2 	/test2 	http://127.0.0.1:8080/ 	/test2/abc/test 	//abc/test
3 	/test3/ 	http://127.0.0.1:8080 	/test3/abc/test 	/test3/abc/test
4 	/test4/ 	http://127.0.0.1:8080/ 	/test4/abc/test 	/abc/test
5 	/test5 	http://127.0.0.1:8080/app1 	/test5/abc/test 	/app1/abc/test
6 	/test6 	http://127.0.0.1:8080/app1/ 	/test6/abc/test 	/app1//abc/test
7 	/test7/ 	http://127.0.0.1:8080/app1 	/test7/abc/test 	/app1abc/test
8 	/test8/ 	http://127.0.0.1:8080/app1/ 	/test8/abc/test 	/app1/abc/test
9 	/ 	http://127.0.0.1:8080 	/test9/abc/test 	/test9/abc/test
10 	/ 	http://127.0.0.1:8080/ 	/test10/abc/test 	/test10/abc/test
11 	/ 	http://127.0.0.1:8080/app1 	/test11/abc/test 	/app1test11/abc/test
12 	/ 	http://127.0.0.1:8080/app2/ 	/test12/abc/test 	/app2/test12/abc/test

    Note: When you have a trailing / in the proxy_pass url then the url mentioned in the location block is removed from the actual url starting and rest of the pass is sent to the proxied server as seen in Case #4

