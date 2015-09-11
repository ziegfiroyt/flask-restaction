# Flask-Restaction

### you can do this Easily

- Create restful api 
- Validate inputs and Convert outputs
- Authorize and Permission control
- Auto generate res.js


### Install
    
    pip install flask-restaction


### Build Document

    cd docs
    make html

### Readthedocs

http://flask-restaction.readthedocs.org/zh/latest/

## Quickstart

#### A minimal Flask-Restaction API looks like this:

```python
from flask import Flask
from flask_restaction import Resource, Api

app = Flask(__name__)
api = Api(app)


class Hello(Resource):
    schema_inputs = {
        "get": {
            "name": {
                "desc": "you name",
                "required": True,
                "validate": "safestr",
                "default": "world"
            }
        }
    }

    def get(self, name):
        return {"hello": name}

api.add_resource(Hello)
api.gen_res_js()

if __name__ == '__main__':
    app.run(debug=True)

```
save this as `hello.py`, then run it: 

    $ python hello.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

then open browser, visit `http://127.0.0.1:5000/hello`
you will see: 

    {
      "hello": "world"
    }

if you visit `http://127.0.0.1:5000/hello?name=kk`
you will see: 

    {
      "hello": "kk"
    }

if you visit `http://127.0.0.1:5000/hello?name=kk`
you will see: 

    {
      "hello": "kk"
    }

### Use res.js

Les's write test.html and save it in static folder

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>test res.js</title>
    <script type="text/javascript" src="/static/res.js"></script>
    <script type="text/javascript">
    function send() {
        var name = document.getElementById("name").value;
        if (name && name != "") {
            var data = {
                name: name
            };
        }
        res.hello.get(data, function(err, value) {
            msg = JSON.stringify(err || value)
            document.getElementById("message").innerText = msg;
        });
    }
    </script>
</head>
<body>
    <input id="name" type="text" placeholder="you name">
    <p id="message"></p>
    <button onclick="send()">GetHello</button>
</body>
</html>
```
then open browser, visit `http://127.0.0.1:5000/static/test.html`

have a try, notice schema_inputs's `"validate": "safestr"`

if you input some unsafe strings, such as: 

`<script type="text/javascript">alert("haha")</script>`

then you inputs will be escape to avoid attack:

`{"hello":"&lt;script type=&#34;text/javascript&#34;&gt;alert(&#34;haha&#34;)&lt;/script&gt;"}`

#### res.js

look at this:

```javascript
res.hello.get(data, function(err, value) {
    msg = JSON.stringify(err || value)
    document.getElementById("message").innerText = msg;
});
```

now we can use `res.resource.action(data, function(err, value))` to access resources provided by rest api.

`resource` is resource's name, such as 'hello',

`action` is ... such as `get`,`post` ... not only http method, `getlist`,`upload` is ok.

### More about validate

see <https://github.com/guyskk/validater>

### Authorize and Permission control

#### authorize

flask_restaction use 'json web token' for authorize.
see <https://github.com/jpadilla/pyjwt>

you should config `RESOURCE_JWT_SECRET` in `app.config`,

defalut value is `"DEFAULT_RESOURCE_JWT_SECRET"`

you can config `RESOURCE_JWT_ALG` in `app.config`,

defalut value is 'HS256'

token store in `request.headers.["Authorization"]`

you can access auth info by `request.me`, it's struct is:

    {
        "id":user_id, 
        "role":user_role
    }

and you should add `Authorization` header to response after user login, 

it's value can be generate by `api.gen_token(self, me)`

Note:

res.js will auto add `Authorization` header to request if needed, and will auto save `Authorization` token to localstroge when recive `Authorization` header


#### Permission 权限分配表

`permission.json` should be saved in root path of you flask application

权限按role->resource->action划分

JSON 文件格式
```json
{
    "role/*": {
        "*/resource*": ["get", "post"],
        "resource": ["action", ...]
    },
    ...
}
```
- role为`*`时，表示匿名用户的权限。
- resource为`*`时，表示拥有所有resource的所有action权限，此时actions必须为`[]`且不能有其他resource。
- resource为`resource*`时，表示拥有此resource的所有action权限，此时actions必须为`[]`。
- role和resource（除去`*`号）只能是字母数字下划线组合，且不能以数字开头。


### Next Todo

- tests 
- support blueprint
- document
- ...
