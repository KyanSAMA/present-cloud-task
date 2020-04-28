# 14 RESTful和HTTP

## 14.1 任务描述

### 14.1.1 任务介绍

实现单页面应用中实现HTTP拦截器。

### 14.1.2 任务要求

- 了解RESTful
- HTTP拦截器的实现
  - 统一处理服务器的域名
  - 统一对服务器的响应进行错误处理
  - 在HTTP的Header中添加Token

## 14.2 工作指导说明

### 14.2.1 RESTful

RESTFUL是一种网络应用程序的设计风格和开发方式，基于HTTP，可以使用XML格式定义或JSON格式定义。RESTFUL适用于移动互联网厂商作为业务使能接口的场景，实现第三方OTT调用移动网络资源的功能，动作类型为新增、变更、删除所调用资源。  
> [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)  
> [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

资源请使用名词并且使用复数，不要使用动词。例如：dictionaries,用POST, DELETE, PUT, GET等http方法，对这个资源进行增删改查。不要使用addDictionary、deleteDictionaries等。  
根据Id获取或者删除某个资源，建议使用：dictionaries/1。  
分页、排序、查询，建议使用传统的链接带参数的方式，例如：dictionaries?offset=20&limit=10，第3页，每页显示10条。dictionaries?nameOrCode=sex。

### 14.2.2 environment设置

#### 开发环境、测试环境和生产环境

修改 src/environments/environment.ts

```ts
export const environment = {
  production: false,
  serverUrl: 'http://127.0.0.1:8000'
};
```

```bash
ng serve
```

修改 src/environments/environment.proc.ts

```ts
export const environment = {
  production: true,
  serverUrl: 'http://www.czb.xyz/api'
};
```

```bash
ng build
```

#### 创建服务

对数据的增删改查的操作都需要访问服务器，建议把这些数据访问的相关代码都放在单独的服务中。  
DictionaryService虽然会被DictionaryList、DictionaryEdit等其它组件使用到，但只是在Dictionary频道中复用，因此不要创建在shared目录中，直接创建在Dictionary根目录中，或者在Dictionary目录中创建shared文件夹。

```ts
@Injectable()
export class DictionaryService {
  url = environment.serverUrl + '/dictionaries';
  constructor(private http: HttpClient) { }

  search(nameOrCode: string): Observable<any> {
    nameOrCode = nameOrCode.trim(); // 上课演示时漏了去除左右空格
    if (!nameOrCode || nameOrCode.length === 0) {
      throw new Error('search参数错误');
    }
    console.log('search url:', this.url);
    // const newUrl = this.url + `?condition=${nameOrCode}`;
    const options = new HttpParams().set('condition', nameOrCode);
    return this.http.get(this.url, {params: options}).pipe(catchError((error: HttpErrorResponse) => {
      console.log('search', error);
      return of(error);
    }));
  }
}
```

继续编写其他的增删改查方法时，对服务器响应的处理都是一摸一样。可以封装一个方法实现。

### 14.2.3 http拦截器

类似上面的案例中，其他http相关的服务中都存在需要传递服务器端链接，需要传递token，处理服务器响应返回的结果。可以在http的拦截器中统一处理。  
在core目录中创建http文件夹。

```bash
ng g interceptor auth
ng g interceptor url
ng g interceptor error
```

#### 处理链接的拦截器

```ts
intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    let url = request.url;
    if (!url.startsWith('https://') && !url.startsWith('http://')) {
      url = environment.serverUrl + url;
    }
    console.log(url);
    const newRequest = request.clone({ url });
    return next.handle(newRequest);
  }
```

#### 处理token的拦截器

```ts
intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const authToken = 'my-authorization-token'; // this.auth.getAuthorizationToken();
    const authReq = request.clone({
      headers: request.headers.set('Authorization', authToken)
    });
    // const authReq = request.clone({ setHeaders: { Authorization: authToken } });
    return next.handle(request);
  }
```

#### 统一异常处理的拦截器

```ts
handleError(error: HttpErrorResponse): Observable<any> {
    console.log('handleError');
    switch (error.status) {
      case 200:
        break;
      case 401:
        break;
      case 404:
        break;
      case 403:
        break;
      case 500:
        break;
      default:
        break;
    }
    return of(error);
  }
  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    // return next.handle(request).pipe(catchError((err: HttpErrorResponse) => {
    //   return this.handleError(err);
    // }));
    return next.handle(request).pipe(catchError((error: HttpErrorResponse) => this.handleError(error)));
  }
```

在拦截器中使用第三方的服务，需要用到Injector。

```ts
// 在构造函数中依赖注入
constructor(private injector: Injector) { }
```

```ts
this.injector.get(Router).navigateByUrl('/passport/login'));
```

## 14.3 产品要求

无

## 14.4 工作要求

无
