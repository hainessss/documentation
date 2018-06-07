# nextgen-utils

A utility repo for next gen apps.

## Installation

Install with yarn:

```
npm add --save nextgen-utils
```

## HSClient

An HTTP client aimed at providing a simple and efficient way of communicating with hotschedule’s APIs from the frontend. 

### Usage

```
import { HSClient } from 'nextgen-utils';

const client = new HSClient();

```

A client can be instantiated with the following options.

```
{
    baseUrl: "https://api.bodhi-dev.io", 
    namespace: "namespace_name", 
    subDomain: "api", 
    basePath: "/resources"
}
```

**baseUrl** - is the root target URL.  

**namepace** - name of target namespace.

**subDomain** - will overwrite or add a sub domain to the base URL

**basePath** - any static path that exists after the namespace name in the URL.  Defaults to "/resources".

### Methods

#### setEnv(config<Object>)


baseUrl setter method. Takes a config object. 

```
const config = {
  "env": "dev",
  "dev": "https://api.bodhi-dev.io",
  "qa": "https://api.bodhi-qa.io",
  "stg": "https://api.bodhi-stg.io",
  "prd": "https://api.bodhi.space"
}

client.setEnv(config);

console.log(client.baseUrl)
// --> https://api.bodhi-dev.io
```

#### setBasePath(path<String>)

basePath setter method.

#### setNamespace(namespace<String>)

namespace setter method.

```
 client.setNamespace('namespace_name')
```

#### setSubDomain(subDomain<String>)

subDomain setter method.

#### fetch(url<String>, method<String>, body<Object>)

Sends an HTTP request. Generally will be called through other methods.

**url** - target url 

**method** - HTTP action

**body** - HTTP body object

#### get(type<String>, overrides<Object>)

Sends and HTTP GET request

**type** 
- target type. For a full list of types in a namespace go to https://tools.bodhi-dev.io/types. 

**overrides** 
- the last parameter takes an object that can override the client's namespace, baseUrl, basePath, and subDomain properties. In addition, you can set page and limit url query parameters in the ovverides object of get requests. 

```
const client = new HSClient();

client.get('type');
// --> https://api.bodhi-dev.io/namespace_name/resources/type
```

You can send get requests to different domains by using the absolute URL

```
client.get('http://www.google.com');
// --> http://www.google.com
```

You append or prepend additional paths to the type.

```
client.get('type/count');
// --> https://api.bodhi-dev.io/namespace_name/resources/type/count

client.get('/type/count');
// --> https://api.bodhi-dev.io/namespace_name/resources/type/count

client.get('/resources/type/count');
// --> https://api.bodhi-dev.io/namespace_name/resources/type/count
```

You can send get requests to a relative path by including a "." at the beginning.  

```
client.get('./config.json');
// --> localhost:8080/config.json
```


You can specify page and result limits.  

```
client.get('Store', {page: 1, limit: 25});
// --> https://api.bodhi-dev.io/namespace_name/resources/Store?paging=page:1,limit:25
```

#### count(type<String>, overrides<Object>)

Sends and HTTP GET request to the '/count' endpoint for the given resource.

**type** 
- target type. For a full list of types in a namespace go to https://tools.bodhi-dev.io/types. 

**overrides** 
- the last parameter takes an object that can override the client's namespace, baseUrl, basePath, and subDomain properties. In addition, you can set page and limit url query parameters in the ovverides object of get requests. 

```
const client = new HSClient();

client.count('type');
// --> https://api.bodhi-dev.io/namespace_name/resources/type/count
```


you can use query parameters like normal. 

```
const client = new HSClient();

client.count('type?where={"attr": "equals this"}');
// --> https://api.bodhi-dev.io/namespace_name/resources/type/count?where={"attr": "equals this"}'
```

#### getAll(type<String>, overrides<Object>)

Will get all of a given resource in the database.

**type** 
- target type. For a full list of types in a namespace go to https://tools.bodhi-dev.io/types. 

**overrides** 
- the last parameter takes an object that can override the client's namespace, baseUrl, basePath, and subDomain properties. In addition, you can set page and limit url query parameters in the ovverides object of get requests. 

```
const client = new HSClient();

client.getAll('type');
// --> will aggregate the results of the following
// --> https://api.bodhi-dev.io/namespace_name/resources/type?paging=page:1,limit:100
// --> https://api.bodhi-dev.io/namespace_name/resources/type?paging=page:2,limit:100
// --> https://api.bodhi-dev.io/namespace_name/resources/type?paging=page:3,limit:100
//etc..
```


you can use query parameters like normal. 

```
const client = new HSClient();

client.count('type?where={"attr": "equals this"}');
// --> https://api.bodhi-dev.io/namespace_name/resources/type/count?where={"attr": "equals this"}'
```


#### post(type<String>, body,Object>, overrides<Object>)

Sends and HTTP POST request

**type** - target type or path. 

**body** - HTTP body object

**overrides** - the last parameter takes an object that can override the client's namespace, baseUrl, basePath, and subDomain properties.

#### put(type<String>, body,Object>, overrides<Object>)

Sends and HTTP POST request

**type** - target type or path. 

**body** - HTTP body object

**overrides** - the last parameter takes an object that can override the client's namespace, baseUrl, basePath, and subDomain properties.

#### upsert(type<String>, body,Object>, overrides<Object>)

Sends and HTTP POST request

**type** - target type or path.

**body** - HTTP body object

**overrides** - the last parameter takes an object that can override the client's namespace, baseUrl, basePath, and subDomain properties.

#### query()

Returns a bodhi-query-builder object. For more docs on bodhi-query-builder visit https://bitbucket.org/redbookplatform/bodhi-query-builder

```
const query = client.query()
                .select('fields, to, return')
                .from('TypeName')
                .where({
                    attr: 'equals this'
                });

client.get(query.toUri())
// --> https://api.bodhi-dev.io/namespace_name/resources/TypeName?fields=fields,to,return&where={"attr": "equals this"}
```

#### me()

Sends a get request to /me which return user information.

```
client.me()
// --> https://api.bodhi-dev.io/me
```

#### asURI()

Returns a new instance of HSClient that returns URI's instead of making request to them.

```
client.asURI().get('Type')
// returns https://api.bodhi-dev.io/namespace_name/resources/Type
```

#### vertx(name<String>)

Returns a new instance of HSClient that changes the subDomain and basePath to point to our vertx services.

**name** - will get appended to the basePath ('/controllers/vertx/{name}' ).

```
client.vertx('upload').post('path/to/file.jpeg')
// --> https://api.bodhi-dev.io/namespace_name/controllers/vertx/Type/upload/path/to/file.jpeg
```

#### applyMiddleware(middlewares<Array>)

Overwrites the fetch function to augment functionality. 

**middlewares** - an array of middleware functions.

## addRxjsMiddleware

Augments HSClient fetch method to return observable.

## addLoggerMiddleware

Augments HSClient to log the request to the console.







## Testing

From the repo root:

```
npm test
```

## Dev Notes

You can write your own middleware with the following format.  

```
const newMiddleWare = (client) => {
        return (next) => {
            return (url, method, body) => {
                return next(url, method, body);
            }
        }
    };
```



## Contributing


### Local
To contribute, please create a new local branch with the same name as your JIRA ticket (e.g NGINTW-9999).  When commiting, message format shoudld look like this:

```
[<JIRA ticket>] <Your commit message here> 
```


Here's an example:

```
[NGINTW-9999] It's over 9000!!!.
```


If typing the JIRA ticket is too tedious, git can automatically prepend the ticket number to your commit message if you symlink the `prepare-commit-msg` script in `gitHooks`.  

Run `setup_git_hooks.sh` to symlink hooks in `gitHooks` folder.


### Pull Request
When code looks good, push your branch and make a PR at:

https://github.com/hotschedules/nextgen-utils/pulls