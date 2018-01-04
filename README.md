自建简易laradock
## 创建容器以及配置
laravel项目在mydock同级mydock_app下，名称为mylaravel,数据库名称为mylaravel,root密码为root,redis密码为redis.  
大部分操作在容器内进行.

### 创建容器：在docker-compoer.yml同级目录下执行
```
cp env-example .env //配置文件
docker-compose up -d nginx mysql redis
```
### 然后进行redis/mysql/laravel配置，首先redis设置密码
```
docker-compose exec redis redis-cli
//首先确认redis是否为无密码状态
set aa 'aa' //返回OK,表示当前redis没有使用密码
config set requirepass redis //设置密码为redis,返回结果为OK
get aa  //返回错误提示(error) NOAUTH Authentication required. 
auth redis //输入之前设置的密码,返回结果为OK
get aa //返回 'aa' ，正常获取结果，密码设置成功
```
设置完毕退出容器进行下一步。

### 在Mysql创建项目数据库,登录密码在.env,这里设置为root
```
docker-compose exec mysql mysql -uroot -proot
show databases;//检查当前数据库，如果已经有数据库则删除
create database mylaravel; // 创建laravel数据库,简要创建
exit;
```
退出这个容器。
### 然后进入workspace创建laravel以及进行配置，生成用户认证
```
docker-compose exec workspace bash
ls //检查当前目录状态，一般为空
composer create-project laravel/laravel mylaravel //创建laravel项目，名称为mylaravel,创建完毕之后进入项目目录
cd mylaravel
ls -al //检查是否有.env文件，如果没有cp env.example .env
vi .env //容器中没有vim，使用vi编辑
```
需要修改的部分：  
```
DB_HOST=mysql
DB_DATABASE=mylaravel
DB_USERNAME=root
DB_PASSWORD=root

SESSION_DRIVER=redis

REDIS_HOST=redis
REDIS_PASSWORD=redis

```
保存退出。  
### 然后生成用户认证等。
```
php artisan session:table //生成session table,不执行也可
php artisan make:auth //生成用户认证，这步会生成2个迁移文件
php artisan migrate //执行迁移，完成后会在数据库中新增4张表，2张用户认证，一张session，一张迁移管理
chmod -R 777 storage // storage目录给予可写权限，省事操作777
composer require predis/predis //有使用redis必须安装
```
完毕后退出容器。 
### 修改nginx/sites/default.conf：
```
server_name localhost;
root /var/www/public; //修改这一行为下一行
root /var/www/mylaravel/public;
```
然后重启docker 的nginx:
```
docker-compose up -d nginx
```
完毕后退出容器。  

## 测试项目运行：
 - 访问localhost.能看见laravel首页视图，右上角有Login和Register。  
 - 点击register,填入资料，提交，页面会跳转到/home并提示You are logged in! 
 - 点击右上角用户名选择logout,跳回到laravel首页。
 - 进入redis容器，如果指令中不带密码，务必首先执行auth xxx。然后keys *,如果输出内容中有带'laravel_cache'说明redis有发挥作用
 - 访问8081端口，显示百度首页

