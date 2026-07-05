# 安全与防护

## 输入过滤

### 全局过滤（filter）

`app/Request.php` 已配置默认过滤规则：
1. `trim()` - 去除两端空格
2. `strip_tags()` - 过滤 HTML/PHP 标签
3. `htmlspecialchars()` - 特殊字符转实体

```php
// 还原过滤后的数据
htmlspecialchars_decode_improve($string);

// 改变过滤规则
$this->request->filter('strip_tags');
```

### 反 XSS（clean_xss）

```php
// 过滤 XSS 攻击代码
$clean = clean_xss($harmString);

// 设置请求输入过滤
$this->request->filter('clean_xss');
$data = $this->request->post();
```

基于 `voku/anti-xss` 实现，用于富文本数据过滤。

## 线上安全防范

### 1. 自定义后台入口

v2.0+ 支持自定义后台入口，建议在开发环境就配置。

### 2. 禁用危险函数

建议禁用：`eval`、`exec`、`shell_exec`、`system`

### 3. 关闭调试

将 `.env` 文件中的 `APP_DEBUG` 设为 `false` 或删除 `.env` 文件。

### 4. 目录权限

```bash
# 修改所有者
sudo chown www:www /www/wwwroot/example.com -R

# 整个站点只读
sudo chmod 555 /www/wwwroot/example.com -R

# 以下目录增加写权限
sudo chmod u+w /www/wwwroot/example.com/runtime -R
sudo chmod u+w /www/wwwroot/example.com/public/storage -R
```

### 5. 防止浏览器预览脚本

```nginx
# Nginx
location ~ .*\.(txt|doc|pdf|rar|gz|zip|docx|exe|xlsx|ppt|pptx)$ {
    add_header Content-Disposition attachment;
}
```

### 6. 禁止上传目录执行代码

```nginx
# Nginx
location ~ ^/storage/.*\.(php|php5|jsp)$ {
    deny all;
}
```

### 7. XSS 防范

- 前端尽量不使用 `v-html`
- 过滤所有用户输入

### 8. CSRF 和会话劫持

- 后台不使用 Cookie 和 Session
- 如需使用 Cookie，开启 httponly

### 9. HTTPS

启用 HTTPS 加密传输。

## Token 安全

- 自动携带 token
- 无感刷新 token
- Token 存储在 localStorage
- 后台和前台使用独立的 token 机制

## 验证码

### 普通验证码

```php
use ba\Captcha;

// 生成
$captcha = new Captcha($config);
return $captcha->entry($captchaId);

// 验证
$captcha = new Captcha();
if (!$captcha->check($code, $captchaId)) {
    $this->error('验证码错误！');
}
```

### 点选文字验证码

```php
use ba\ClickCaptcha;

$captcha = new ClickCaptcha();
if (!$captcha->check($captchaId, $captchaInfo)) {
    $this->error('验证码错误！');
}
```
