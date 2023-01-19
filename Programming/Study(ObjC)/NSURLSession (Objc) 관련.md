# NSURLSession (Objc) 관련

### 다른거 Import 하거나 뭐 할 필요 없이 아래 코드들 사용해서 활용 가능함

## NSString 의 URL 생성 방법
- 코드 참고할 것
    ```objectivec
    - (NSString *)setUrl :(NSString *)city {
        NSString *urlString = @"https://api.openweathermap.org/data/2.5/weather?";
        NSMutableDictionary *urlParameters = [[NSMutableDictionary alloc] init];
        [urlParameters setObject:@"f518a6fc1d40565ebe52af2ab90320e2" forKey:@"appid"];
        [urlParameters setObject:city forKey:@"q"];
        
        NSString *url = urlString;
        
        for (NSString *key in urlParameters) {
            NSString *valueData = [NSString stringWithFormat:@"%@",urlParameters[key]];
            
            url = [url stringByAppendingString:key];
            url = [url stringByAppendingString:@"="];
            url = [url stringByAppendingString:valueData];
            url = [url stringByAppendingString:@"&"];
        }
        url = [url substringToIndex:url.length-1];
        
        return url;
    }
    ```

### 통신부
- 코드 참고할 것
    ```objectivec
    - (void)httpRequest:(NSString *)url {
        NSURL *request = [NSURL URLWithString:url];
        // NSString 으로 받은 url을 NSURL로 변환
        
        NSMutableURLRequest *urlRequest = [[NSMutableURLRequest alloc] initWithURL:request];
        [urlRequest setHTTPMethod:@"GET"];
        [urlRequest setValue:@"application/x-www-form-urlencoded; charset=utf-8" forHTTPHeaderField:@"Content-Type"];
        // Request 처리를 위한 최종 URL(?) 작성
        
        NSURLSession *session = [NSURLSession sharedSession];

        NSURLSessionDataTask *task = [session dataTaskWithRequest:urlRequest completionHandler:^(NSData * data, NSURLResponse * response, NSError * error) {
        
            NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *) response;

            NSDictionary *headerDictionary = [httpResponse allHeaderFields];
            
            NSString *returnData = @"";
            // 이곳에 이제 통신으로 받은 모든 데이터들을 저장 받게 될것

            if (error != nil) {
                @try {
                    returnData = error.localizedDescription;
                } @catch (NSException *exception) {
                    NSLog(@"HTTP Http Request Exception [Error] :: %@", exception);
                }
                NSLog(@"ERROR");
                NSLog(@"%@",returnData);

            } else {
                if (httpResponse.statusCode >=200 && httpResponse.statusCode <= 300) {
                    @try {
                        returnData = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
                    } @catch (NSException *exception) {
                        NSLog(@"Http Request Exception [Response] :: %@", exception);
                    }
                    NSLog(@"SUCCESS");
                    NSLog(@"%@",returnData);

                } else {
                    returnData = [returnData stringByAppendingString:@"State Code Error: "];
                    returnData = [returnData stringByAppendingString:[@(httpResponse.statusCode) stringValue]];
                    returnData = [returnData stringByAppendingString:@" : "];
                    returnData = [returnData stringByAppendingString:[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]];
                    NSLog(@"ERROR2");
                    NSLog(@"%@",returnData);
                }
            }
        }];
        
        [task resume];
    }
    ```