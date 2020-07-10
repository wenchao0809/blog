nginx抓发待cookie

location ^~ /api/ {
            proxy_pass https://business-test.haina.com/api/;
            proxy_set_header Cookie $http_cookie;
            proxy_cookie_domain business-test.haina.com localhost;
            proxy_set_header Host business-test.haina.com;
          }


ie11好像会失效
