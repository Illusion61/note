# 实现登录功能

## 架设网站

#### 前端提交表单

- 发送x-www-form-urlencoded形式的数据:
  - **name**:发送表单时候的**数据项名称**
  - **action**:发送表单的**地址**

```html
<form action='https://manga.sciform.one/api/login/' method="post" id="loginForm">
    <input type="text" id="username" name="username" placeholder="Enter your username" required>
    <input type="password" id="password" name="password" placeholder="Enter your password" required>
    <input type="submit" value="Login">
</form>
```

- 发送json形式的数据:**使用ajax**

```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
	const submit = document.querySelector('.chat-submit input[type="submit"]')
    submit.addEventListener('click',function(){
        $.ajax({
          url:`https://sciform.one/api/chat`,
          type:'GET',
          data:{content:mes},
          success:function(data){
            console.log(data);
          }
        })
      })
</script>
```

#### express

```javascript
const express = require("express")
```

```javascript
const app = express()
//路由&&中间件
app.get('/api/login',(req,res)=>{
    if(req.query.id == 'admin' && req.query.password == 'root')
    	res.send("<h1>Hello</h1>")
})
app.listen(8080,()=>{
    console.log("Server is on. Listening on port 8080")
})
```

```javascript
//获取请求参数:get方法url中参数(http://xxx/api/login?username=admin&password=root)
req.query.参数名
//获取请求参数:post方法body中json
app.post('/api/login',express.json(),(req,res)=>{
    if(req.body['id'] == 'admin' && req.body['password'] == 'root')
        res.send("<h1>Hello</h1>")
})
//获取请求参数:post方法body中的x-www-form-urlencoded(body中也是username=admin&password=root)
//浏览器表单提交默认是x-www-form-urlencoded
const bodyParser = require('body-parser')
app.post('/api/login',bodyParser.urlencoded({extended:true}),(req,res)=>{
    const {username,password} = req.body
    if(username == 'admin' && password == 'root')
        res.send("<h1>Hello</h1>")
})
```

```javascript
res.send('<h1>Hello</h1>')
res.setHeader('Content-Type', 'text/html');
res.redirect('https://manga.sciform.one/artworks/')
```



#### https

```javascript
const fs = require("fs")
const https = require("https")
const options = {
	key:fs.readFileSync('/var/www/credentials/sciform.one/privkey.pem'),
	cert:fs.readFileSync('/var/www/credentials/sciform.one/fullchain.pem')
}
const httpsServer = https.createServer(options,app)
httpsServer.listen(443,()=>{
  console.log("Server is on listening port 443");
})
```

## 连接mysql数据库

```mysql
create table users(
	username varchar(30),
	password varchar(30),
    email varchar(30),
    isadmin TINYINT,
    isexist TINYINT
);
INSERT INTO users VALUES ("admin","root","illusion@sciform.one",1,1);
CREATE INDEX
```

```javascript
const mysql = require("mysql2")
const db = mysql.createPool({
    host:'127.0.0.1',
    user:'root',
    password:"6d2ySW2py",
    database:'illusion'
})

db.query('SELECT * FROM account WHERE username=?',username,(err,results)=>{
    if(results[0].password == req.query.password)......
})
db.query('INSERT INTO users SET ?',{username:username,password:password,email:email,isadmin:0,isexist:1},(err,dataStr)=>{
    if(err)return console.log(err);
    console.log(dataStr);
    if(dataStr.affectedRows === 1){
        res.redirect("http://manga.sciform.one:8080/login")
    }
})
```

## 连接Redis

- 连接

```javascript
const redis = require('redis');
const redisClient = redis.createClient({
	url: 'redis://:6d2ySW2py@107.173.181.210:6379'
});
redisClient.on('connect', () => {
	console.log('Connected to Redis server');
});
redisClient.connect(6379, '107.173.181.210')
```

- 查询数据库,缓存

```javascript
//获取redis中的数据
const resp = await redisClient.get(id)
//数据已经存在,直接返回数据
if(resp)
    res.send(JSON.parse(resp))
//数据不存在,(在mysql中查找数据,并添加到redis中),然后返回数据
else{
    db.query('SELECT * FROM subways WHERE id=?', id, (err, results) => {
        if (err) {
            res.status(500).send('Error');
        } else {
            // 将查询结果存入Redis缓存
            redisClient.set(id, JSON.stringify(results));
            res.send(results);
        }
    });
}
```

- 使用HSET等方法

```javascript
const testData = async()=>{
	await redisClient.HSET('hash','user:illusion',3)
	const num = +await redisClient.HGET('hash','user:illusion')
	await redisClient.HSET('hash','user:illusion',num+1)
}
```

- 注意:
  - 使用方法的时候字母要大写,比如HGET而不是hget
  - **无论是GET还是SET,都必须要await**
  - 注意默认是字符串数据类型,要通过+转化为数字

## 登录自身的逻辑

#### Session

- 概念:session是服务器端存储用户信息的方式,用于保存用户会话信息.在会话结束的时候自动删除
- 步骤:
  - 用户建立会话,进行登录获取cookie或者直接提交cookie,**获取sessionID**(由express自动生成)
  - 以后每次提交请求,**都会加上sessionID**,express根据sessionID**自动获取到对应的session数据**

```javascript
//cookie:{maxAge:15000}默认单位为毫秒(15s)
//如果配置cookie,就默认{maxAge:0}-->即为浏览器关闭就过期,浏览器不关闭就一直继续
app.use(session({
secret:'ajnACNZLS2KBLBIASnckbjalbcbszbcjkbvuiacskcvgaksvxjhsafcvskavdcjhkasbjkwvchkuawjgbZNOIBWCVZKVCKJZGWIAGBSCHXZVBJKUvXAVCKBACB219GBCUY1OGCBAUbicgoaivcbuoqgbcioqg2bdcuoqilnxcboalbcqbcowehgbvczlabklewv709cabBKVWOUCSAEFGV2',
    cookie:{maxAge:15000},
    resave:false,
    saveUninitialized:true
}))
```

```javascript
const session = require("express-session")
app.get('/api/login',(req,res)=>{
    ......
    req.session.user = req.query.username;
    req.session.isLogin = true;
})
const auth = (req,res,next)=>{
    if(!req.session.isLogin)
        return false;
    next();
}
```

#### 注意事项

- 密码用哈希存储
- json字符串必须用英文双引号
- 验证权限:
  - 使用session.isLogin验证
- 登录:
  - 先检测是否有session,如果session.isLogin=true直接重定向
  - 判断用户名密码是否符合格式要求
  - 查询数据库:
    - 如果未激活,不能登录
    - 判断密码是否正确,用户名是否存在
- 注册:
  - 判断用户名,密码,邮箱是否符合格式要求(字数不能超,不能有特殊字符,邮箱和手机符合格式)
  - 判断是否已经有这个用户和邮箱存在(唯一性)
  - 初始状态未激活.先要登陆邮箱激活

## 发邮件

```javascript
const mailTransport = nodemailer.createTransport({
	host : 'smtp.126.com',
	secureConnection: true, // 使用SSL方式（安全方式，防止被窃取信息）
	auth : {
		user : 'jiang_sicheng@126.com',
		pass : 'ZWYAZBASAHHOSHVF'
	},
});
const mailOptions = {
    from: '"song4emotion" <root@sciform.one>', // sender address  
    to: `"${username}" <${email}>, xxxxxxxx@gmail.com`, // list of receivers 接收者地址
    subject: 'Hello friend', // Subject line                      // 邮件标题
    text: 'this is nodemailer test', // plain text body
    html: '<b>Big test</b>' // html body   //邮件内容
};
mailTransport.sendMail(mailOptions,(err,info)=>{
    if (err)
        return console.log(error);
    console.log('Message sent: %s', info.messageId);//成功回调
    console.log('Preview URL: %s', nodemailer.getTestMessageUrl(info));
})
```



# 同步数据

# 限制api访问

## 布隆过滤器



## 限制同一个IP频繁访问

- 注意:由于X-Real-IP和X-Forwarded-For是可以被人为修改的,所以光限制IP还不够

```javascript
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
	windowMs: 1 * 60 * 1000, // 1 分钟
	max: 100,
});
app.use(limiter);// 应用限制中间件到所有路由
```

- 如果

```javascript
// 仅信任受信任代理服务器的 IP 地址或 IP 范围
app.set('trust proxy', '::ffff:18.141.110.6'); // 仅信任回环地址（localhost）的代理
```

