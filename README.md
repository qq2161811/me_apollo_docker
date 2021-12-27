# docker-compose部署多环境apollo及同步配置

## 暴露端口的方式

### 前言

基于以下几点，写下了本文

1. apollo官网docker部署介绍比较模糊
2. 网上其他人部署也大多是抄袭，而且说得也是模糊不清，又或者操作过于繁杂，看不懂。
3. 本人捣鼓了一下，借鉴了很多的文章，终于配置多环境使用已无问题。

### 部署说明

1. 版本：apollo:latest
2. 部署方式：基于源码+docker-compose+mysql实现一键构建镜像部署，前提是必须对docker-compose有一定了解
3. 环境介绍：mac或linux系统机器一台，分别配置dev，fat，uat，pro多环境
4. 同步配置操作语言：php（laravel框架）

### 部署步骤

#### 1. 安装docker

这步忽略，因为经常使用docker懒得再去操作一遍了，自行百度找教程安装，非常方便的，如果连docker都没了解过那下面看的也是一脸懵，除非你有耐心

#### 2. 下载docker-compose资源

```shell
$ git clone http://xxxx
```

#### 3. 启动容器

```shell
docker-compose up -d 
```

第一次执行会比较久，因为需要拉取镜像，所以不要着急，等等就好，还有就是要确认端口未被使用

##### 验证是否启动成功

```shell
# 查看启动的容器
$ docker ps
```

![image-20211224150435616](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224150435616.png)

出现这些容器证明已启动成功

注意：看上图alpine:latest这个容器始终处理Restarting状态这个是正常的，可以无需理会，等下会会在分析docker-compose做详细说明

### apollo多环境验证

#### 1. 确认服务以启动

```shell
# 查看apollo-portal日志或者浏览器是否能打开服务地址
# 1.查看apollo-portal日志
$ docker logs 4c8e39759230
INFO 1 --- [HealthChecker-1] c.c.f.a.portal.component.PortalSettings  : Env revived because env health check success. env: DEV
INFO 1 --- [HealthChecker-1] c.c.f.a.portal.component.PortalSettings  : Env revived because env health check success. env: FAT
INFO 1 --- [HealthChecker-1] c.c.f.a.portal.component.PortalSettings  : Env revived because env health check success. env: UAT
INFO 1 --- [HealthChecker-1] c.c.f.a.portal.component.PortalSettings  : Env revived because env health check success. env: PRO
# 2.浏览器是否能打开服务地址
http://192.168.0.152:8080/
http://192.168.0.152:8081/
http://192.168.0.152:8082/
http://192.168.0.152:8083/
```

![image-20211224154035117](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224154035117.png)

访问服务地址是这样就证明启动成功！

这里我捣鼓了有一阵子启动不成功，这里我遇到了坑

```shel
2021-12-23 18:53:09.730 ERROR 1 --- [HealthChecker-1] c.c.f.a.portal.component.PortalSettings  : Env health check failed, maybe because of meta server down or configure wrong meta server address. env: PRO, meta server address: http://192.168.0.152:8083
com.ctrip.framework.apollo.common.exception.ServiceException: No available admin server. Maybe because of meta server down or all admin server down. Meta server address: http://192.168.0.152:8083
2021-12-23T10:53:09.731747600Z 	at com.ctrip.framework.apollo.portal.component.RetryableRestTemplate.getAdminServices(RetryableRestTemplate.java:228)
2021-12-23T10:53:09.731778200Z 	at com.ctrip.framework.apollo.portal.component.RetryableRestTemplate.execute(RetryableRestTemplate.java:132)
2021-12-23T10:53:09.731817700Z 	at com.ctrip.framework.apollo.portal.component.RetryableRestTemplate.get(RetryableRestTemplate.java:98)
2021-12-23T10:53:09.731839600Z 	at com.ctrip.framework.apollo.portal.api.AdminServiceAPI$HealthAPI.health(AdminServiceAPI.java:43)
2021-12-23T10:53:09.731858200Z 	at com.ctrip.framework.apollo.portal.component.PortalSettings$HealthCheckTask.isUp(PortalSettings.java:148)
2021-12-23T10:53:09.731879300Z 	at com.ctrip.framework.apollo.portal.component.PortalSettings$HealthCheckTask.run(PortalSettings.java:124)
2021-12-23T10:53:09.731895700Z 	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
2021-12-23T10:53:09.731916500Z 	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
2021-12-23T10:53:09.731936300Z 	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
2021-12-23T10:53:09.731952000Z 	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
2021-12-23T10:53:09.731972700Z 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
2021-12-23T10:53:09.731991700Z 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
2021-12-23T10:53:09.732010300Z 	at java.lang.Thread.run(Thread.java:748)
2021-12-23T10:53:09.732047000Z 
2021-12-23T10:53:09.732114700Z 2021-12-23 18:53:09.731 ERROR 1 --- [HealthChecker-1] c.c.f.a.portal.component.PortalSettings  : Env is down. env: PRO, failed times: 135, meta server address: http://192.168.0.152:8083
```

排查：浏览器是否打开能服务地址，如上述错误描述的：http://192.168.0.152:8083

如果能打开可能就是配置的服务地址与对应环境数据库配置的服务地址不一致导致，反复确认，如果不一致那么就把数据库的配置改成一致，不能一边是http://192.168.0.152:8083，另一边却是http://127.0.0.1:8083这样，虽然都能访问，但就是不行，务必一致，如果都没问题那在去百度吧，我也不清楚具体原因了

数据库配置如下

![image-20211224152939572](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224152939572.png)

#### 2. 数据库连接（navicat）

![image-20211224153132247](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224153132247.png)

一键配置的mysql的密码为空，所以密码那行不填就行，如果需要密码请根据自身需求自行配置

### 访问apollo后台

#### 登录

```shell
# 以下三种方式都能打开
http://192.168.0.152:8070/
http://127.0.0.1:8070/
http://localhost:8070/
```

![image-20211224154825283](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224154825283.png)

登录用户名密码：apollo/admin

登录后验证创建用户，看是不是会报错，能创建用户成功就OK了。

#### 查看项目

可以查看实例项目或者自行创建项目

![image-20211224155054269](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224155054269.png)

进入项目后看到配置的4个环境都有就证明成功了！

至此安装全部完成。

### 项目讲解

#### 目录结构

![image-20211224160214912](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224160214912.png)

1. apollo-adminservice-\*和apollo-configservice-\*这些文件夹是保存日志文件
2. build文件夹的apollo-env.properties是保存服务地址
3. database文件夹是保存数据库数据
4. sql文件夹是保存各个环境的数据库表
5. .env文件用于定义docker-compose.yml文件中使用的变量,主要是暴露的端口定义，数据库连接串信息
6. docker-compose.yml文件是执行文件

#### docker-compose讲解

```dockerfile
version: '2'
services:
    # 开发环境  configservice
    apollo-configservice-dev:
        container_name: apollo-configservice-dev
        image: apolloconfig/apollo-configservice
        restart: always
        depends_on:
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_DEV}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${CONFIG_PORT_DEV}
        volumes:
            - ${LOG_BASE_DIR}/apollo-configservice-dev/logs:/opt/logs
        ports:
            - "${CONFIG_PORT_DEV}:8080"

    # 开发环境  adminservice
    apollo-adminservice-dev:
        container_name: apollo-adminservice-dev
        image: apolloconfig/apollo-adminservice
        restart: always
        depends_on:
            - apollo-configservice-dev
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_DEV}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${ADMIN_PORT_DEV}
        volumes:
            - ${LOG_BASE_DIR}/apollo-adminservice-dev/logs:/opt/logs
        ports:
            - "${ADMIN_PORT_DEV}:8090"

    # fat  configservice
    apollo-configservice-fat:
        container_name: apollo-configservice-fat
        image: apolloconfig/apollo-configservice
        restart: always
        depends_on:
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_FAT}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${CONFIG_PORT_FAT}
        volumes:
            - ${LOG_BASE_DIR}/apollo-configservice-fat/logs:/opt/logs
        ports:
            - "${CONFIG_PORT_FAT}:8080"

    # fat  adminservice
    apollo-adminservice-fat:
        container_name: apollo-adminservice-fat
        image: apolloconfig/apollo-adminservice
        restart: always
        depends_on:
            - apollo-configservice-fat
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_FAT}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${ADMIN_PORT_FAT}
        volumes:
            - ${LOG_BASE_DIR}/apollo-adminservice-fat/logs:/opt/logs
        ports:
            - "${ADMIN_PORT_FAT}:8090"

    # uat  configservice
    apollo-configservice-uat:
        container_name: apollo-configservice-uat
        image: apolloconfig/apollo-configservice
        restart: always
        depends_on:
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_UAT}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${CONFIG_PORT_UAT}
        volumes:
            - ${LOG_BASE_DIR}/apollo-configservice-uat/logs:/opt/logs
        ports:
            - "${CONFIG_PORT_UAT}:8080"

    # uat  adminservice
    apollo-adminservice-uat:
        container_name: apollo-adminservice-uat
        image: apolloconfig/apollo-adminservice
        restart: always
        depends_on:
            - apollo-configservice-uat
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_UAT}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${ADMIN_PORT_UAT}
        volumes:
            - ${LOG_BASE_DIR}/apollo-adminservice-uat/logs:/opt/logs
        ports:
            - "${ADMIN_PORT_UAT}:8090"


    # pro  configservice
    apollo-configservice-pro:
        container_name: apollo-configservice-pro
        image: apolloconfig/apollo-configservice
        restart: always
        depends_on:
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_PRO}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${CONFIG_PORT_PRO}
        volumes:
            - ${LOG_BASE_DIR}/apollo-configservice-pro/logs:/opt/logs
        ports:
            - "${CONFIG_PORT_PRO}:8080"

    # pro  adminservice
    apollo-adminservice-pro:
        container_name: apollo-adminservice-pro
        image: apolloconfig/apollo-adminservice
        restart: always
        depends_on:
            - apollo-configservice-pro
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${DATASOURCE_URL_PRO}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
            EUREKA_INSTANCE_IP_ADDRESS: ${IP_ADDRESS}
            EUREKA_INSTANCE_HOME_PAGE_URL: http://${IP_ADDRESS}:${ADMIN_PORT_PRO}
        volumes:
            - ${LOG_BASE_DIR}/apollo-adminservice-pro/logs:/opt/logs
        ports:
            - "${ADMIN_PORT_PRO}:8090"

    # portal
    apollo-portal:
        container_name: apollo-portal
        image: apolloconfig/apollo-portal
        restart: always
        depends_on:
            - apollo-adminservice-dev
            - apollo-adminservice-fat
            - apollo-adminservice-uat
            - apollo-adminservice-pro
            - apollo-db
        environment:
            SPRING_DATASOURCE_URL: ${PORTAL_DATASOURCE_URL}
            SPRING_DATASOURCE_USERNAME: ${DATASOURCE_USERNAME}
            SPRING_DATASOURCE_PASSWORD: ${DATASOURCE_PASSWORD}
        volumes:
            - ${LOG_BASE_DIR}/apollo-portal/logs:/opt/logs
            - ${BUILD_BASE_DIR}/build/apollo-env.properties:/apollo-portal/config/apollo-env.properties
        ports:
            - "8070:8070"
    apollo-db:
        image: mysql:5.7
        container_name: apollo-db
        environment:
            TZ: Asia/Shanghai
            MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
        ports:
            - "13306:3306"
        volumes:
            - ./sql:/docker-entrypoint-initdb.d
            - ./database:/var/lib/mysql
        volumes_from:
            - apollo-dbdata
        restart: always
    apollo-dbdata:
        image: alpine:latest
        container_name: apollo-dbdata
        volumes:
            - /var/lib/mysql
        restart: always
```

1. version版本是2，因为volumes_from在3被废弃，而且3不是2的升级版，只是运用不一样，所以无需纠结

2. 每个容器下的image使用的是dockerhub的镜像

3. depends_on是每个环境需要依赖的容器

4. environment参数是参考dockerhub对应镜像启动实例里面的参数

   ![image-20211224161518681](docker-compose%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%8E%AF%E5%A2%83apollo%E5%8F%8A%E5%90%8C%E6%AD%A5%E9%85%8D%E7%BD%AE.assets/image-20211224161518681.png)

5. volumes是宿主机与容器的文件挂载（宿主机就是自己买云机器或电脑）

6. ports是宿主机与容器的端口映射

7. mysql容器的volumes_from我也不大清楚是什么意思，反正mysql挂载了apollo-dbdata这个容器就会自动帮我们创建好sql文件下的数据库，所以数据库不用自己另外去创建，执行docker-compose都会帮你弄好，你只需要去修改下配置就行，简直是保姆级的docker-compose

8. apollo-dbdata应该是执行数据库表操作的



## PHP接入apollo

### 更新.env文件配置

```php
# 更新.env文件配置，laravel的写法，这个是artisan执行，可以借鉴
# 1.创建.env这个就配置文件，一般框架都自带了
# 2.创建UpdateApollo.php文件
<?php

namespace App\Console\Commands;

use ApolloSdk\Config\Client;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Storage;

define('ENV_DIR', base_path().DIRECTORY_SEPARATOR.'env');

define('ENV_TPL', ENV_DIR.DIRECTORY_SEPARATOR.'.env_tpl.php');

define('ENV_FILE', base_path().DIRECTORY_SEPARATOR.'.env');

class UpdateApollo extends Command
{

    protected $name = 'update:apollo';//命令名称

    protected $description = '更新apollo配置'; // 命令描述，没什么用

    public function handle(): void
    {
        $client = new Client([
            'config_server_url' => 'http://192.168.0.152:8080',//Apollo配置服务的地址，必须传入这个参数
            'secret' => '8de9b7ab3f2c46fd8ea784559c9d3de3',//密钥，如果配置了密钥可以通过
        ]);

        $appId         = 'SunziLocal';
        $namespaceName = 'application';

        $config = $client->getConfig($appId, $namespaceName, true);
        if ($config === false) {             //获取配置失败
            print_r($client->getErrorInfo());//当产生curl错误时此处有值
            print_r($client->getHttpInfo()); //可能不是产生curl错误，而是阿波罗接口返回的http状态码不是200或304
        } else {
            $this->line('开始同步apollo配置');
            // env配置
            ob_start();
            include ENV_TPL;
            $env_config = ob_get_clean();
            file_put_contents(ENV_FILE, $env_config);
            $this->line('同步完成');
        }
    }
}

# .env_tpl.php文件
<?php
$envs = '';
foreach ($config as $key => $value) {
    $envs .= "$key={$value}".PHP_EOL;
}
echo $envs;

```

### 更新config文件配置

```php
# 更新laravel的config文件配置，laravel的写法，这个是artisan执行，可以借鉴
# 1.config/apollo.php
<?php

declare(strict_types = 1);

return [
    'sync' => [
        'redis' => false,
        'file' => true,
    ]
];

# 2.创建UpdateApollo.php文件
<?php

namespace App\Console\Commands;

use ApolloSdk\Config\Client;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Storage;

class UpdateApollo extends Command
{

    protected $name = 'update:apollo';//命令名称

    protected $description = '更新apollo配置'; // 命令描述，没什么用

    /**
     * @throws \Exception
     * @author ZhongYu 2021/12/24 10:14 AM
     */
    public function handle(): void
    {
        $client = new Client([
            'config_server_url' => 'http://192.168.0.152:8080',//Apollo配置服务的地址，必须传入这个参数
            'secret' => '8de9b7ab3f2c46fd8ea784559c9d3de3',//密钥，如果配置了密钥可以通过
        ]);

        $appId         = 'SunziLocal';
        $namespaceName = 'application';

        $config = $client->getConfig($appId, $namespaceName, true);
        if ($config === false) {             //获取配置失败
            print_r($client->getErrorInfo());//当产生curl错误时此处有值
            print_r($client->getHttpInfo()); //可能不是产生curl错误，而是阿波罗接口返回的http状态码不是200或304
        } else {
            $this->line('开始同步apollo配置');
            // config配置
            $this->doSync($config);
            $this->line('同步完成');
        }
    }

    protected function doSync($cfg)
    {
        try {
            $cfg = array_map(function ($value) {
                if ($row = json_decode($value, true)) {
                    return $row;
                }
                return $value;
            }, $cfg);

            $items = [];

            foreach ($cfg as $key => $value) {
                data_set($items, $key, $value);
            }

            foreach ($items as $k => $item) {
                $this->line('Saving ['.$k.']');
                $this->save($k, $item);
            }

        } catch (\Exception $ex) {
            $this->error($ex->getMessage());
        }
    }

    protected function save($fileName, $item)
    {

        if(config('apollo.sync.redis', false)){
            cache()->tags('apollo')->forever($fileName, $item);
            $this->line('Saving To Redis '.$fileName);
        }

        if(config('apollo.sync.file', false)){

            $this->line('Saving To File '.$fileName);

            $fileName = 'apollo/' . $fileName . '.php';
            ksort($item);
            $content = implode("\r\n", [
                "<?php",
                "return",
                var_export($item, true) . ';'
            ]);

            Storage::disk('config')->put($fileName, $content);
        }

        $this->line('==================');
    }
}

# 3.config文件夹创建个apollo的文件夹和对应的配置文件，具体只能看代码做调整了
# 4.apollo后台配置key为文件名，value为json字符串

```



















