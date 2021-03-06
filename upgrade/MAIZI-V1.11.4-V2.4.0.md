## SAAS V1.11.4-V2.4.0升级文档  190414
### 调整说明
- 本次saas-admin新增advance页面进行配置sms、email通知，在该页面配置之后，config/prod.yaml里sms、email字段作废
- saas-admin增加权限config/prod.yaml里需增加内容(详见第四部)
- 新增观察钱包（详见第五步）
- saas-admin更新部署步骤（详见第六步）
- 自审计
### 部署步骤
#### 一、拉取saas最新包
##### saas后端
- jadepool-saas-backend-V2.4.0-ubuntu.tar.gz
##### saas前端
- jadepool-saas-frontend-V2.4.0-math-ubuntu.tar.gz
##### saas admin
- jadepool-saas-admin-V2.4.0-prefix-selfcustody-ubuntu.tar.gz
#### 二、停用saas服务、备份saas postgres数据库、role service服务不用停
#### 三、更新saas/saas-admin权限
```bash
git clone https://github.com/nbltrust/saas-doc.git
cd saas-doc
git pull
git checkout V2.4.0
cd ./initSaasRole
vim upsert.js   查看第31行是否是本机ip地址，要换成role service所在服务器IP地址
npm i
node upsert.js  该node执行完后执行下一行的
node upsert.js saas-admin
```
#### 四、文件替换
1.从V1.11.4版本的jadepool-saas-backend中拷贝config、pm2、locales文件到V2.4.0版本的jadepool-saas-backend
- config/prod.yaml里superadmin字段需增加内容
```bash
access_control_enable: true
service_role_url: "http://127.0.0.1:6666"   ip为该机ip地址
service_name: "jadepool-saas-admin"
```
- config/prod.yaml里需加自审计字段
```bash
audit_database:    (这些信息参考配置文件database字段)
  host: 127.0.0.1
  port: 5432
  name: xxx
  user: xxx
  pass: xxx
```
#### 五、激活address book步骤
1. 因为此功能依赖hub jadepool-addrbook服务，需要保证hub addrbook服务启动正常
2. 注意hub addrbook服务启动时同样需要配置JP_SECRET环境变量
3. 运行address book tool(以添加ETH为例)
```bash
cd ./jadepool-saas-backend
./bin/tools/addrbook -op chainList    该命令可查看可以添加哪些链
./bin/tools/addrbook -chain ETH -token ETH -op tokenAdd -decimal 18      该命令添加具体的币
```
#### 六、修改nginx saas.conf配置指向最新的安装包路径
#### 七、saas-admin部署步骤更新

1. 修改请求的后端服务url

假设saas-superadmin服务对应的url为https://saas-superadmin.custody.net
```bash
cd jadepool-saas-admin

vim index.html
# 修改href
<meta id="backend_host" href=""/>改为
<meta id="backend_host" href="https://saas-superadmin.custody.net"/>
```
#### 八、数据库更新
1. 插入hubwallet
```SQL
  INSERT INTO hub_wallets (created_at, updated_at, hub_wallet_id) VALUES ('2020-04-14 09:56:28.136873+00', '2020-04-14 09:56:28.136873+00', 'default')
```
2. 设置刚才添加的hub_wallet的wallet_blockchain_list为这热主钱包所有链的id数组（例如：{1,2,3}）， wallet_asset_list为这热主钱包所有币的name数组（例如：{BTC,ETH}）   注意：查看hub default钱包下开启了哪些链和哪些币
3. 修改companies列表里hub_wallet_id为空的改为default, share_hub_wallet为true
4. 所有的apps里hub_app_id为空的改为prod.yaml里jadepool_appid的值

#### 九、重启nginx    service nginx restart

#### 十、启动saas服务

#### 十一、saas-admin jadepool页同步一下链

#### 十二、saas-admin advance页面内容更改
- advance页面配置sms、email内容
- advance页面Domain Settings有可能会读取不到配置文件，可手动添加配置文件的saas_web_url对应的域名










