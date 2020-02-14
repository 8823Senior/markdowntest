# 易方科技客服系统
**关于该客服系统**  

该客服系统基于第三方开源客服系统（春松客服）做修改，未采用前、后端分离

**目录**  

[春松客服说明](#aboutcskefu)  
[部署方案](#deployscheme)  
[For后端开发人员](#forJavaDeveloper)  
&nbsp;&nbsp;[后台操作手册](#forJavaDeveloper_manual)  
&nbsp;&nbsp;[访客端端主要请求接口逻辑](#forJavaDeveloper_customerApiOrder)  
&nbsp;&nbsp;[主要数据库表及关系](#forJavaDeveloper_db)  
[For前端对接](#forFrontDeveloper)  

## <span id="aboutcskefu">关于春松客服的说明</span>

**开源项目地址：** [https://github.com/chatopera/cosin](https://github.com/chatopera/cosin)

**开发环境搭建：** [https://github.com/chatopera/cosin/wiki/春松客服：开发环境](https://github.com/chatopera/cosin/wiki/%E6%98%A5%E6%9D%BE%E5%AE%A2%E6%9C%8D%EF%BC%9A%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)

**基于Docker的部署方案：** [https://github.com/chatopera/cosin/wiki/服务器部署](https://github.com/chatopera/cosin/wiki/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2)

##  <span id="deployscheme">普通部署方案</span>
### 依赖中间件
* redis:5.0.5  
* mysql:5.7.27  
* activemq:5.14.3  
* elasticsearch:2.4.6

### 部署方式&步骤
1. 默认端口 **8035**，WebSocket通信端口 **8036**
1. 因为有内嵌Tomcat可以直接java -jar 启动war包
1. maven打war包 ``mvn clean install -Dmaven.test.skip=true ``，并上传war包到服务器
1. 可通过 ``nohup java -jar kefu.war & `` 启动部署项目，这种方式则日志全部输出到 war包同目录的文件nohup.out中，若想将日志分开则不要nohup
1. 是否启动成功可以通过 **nohup.out** 日志文件查看，启动大概需要40秒

## <span id="forJavaDeveloper">For后端开发人员</span>
### 开发说明
**后端向前端页面传参方式**  
ModelAndView  
ModelMap  

**html页面**  
页面大量使用了FreeMarker模板语言来处理参数和页面动态展示，若要改页面则需要对FreeMarker有一定了解

### <span id="forJavaDeveloper_manual">客服系统后台操作手册</span>
**管理员账号**  
默认账号与密码： admin/admin1234  
#### <span id="forJavaDeveloper_manual_addWebSite">新增一个接入网站需要做哪些操作</span>
1. 登录管理员账号->系统管理  
1. **创建组织机构**  
  + 菜单：用户和组->组织机构，在机构树中，在“组织机构”这个根节点下创建一个机构（即部门）  
  + 特别注意 ***要选择正确的上级机构（无特殊要求则都选择跟节点） 以及 勾选启用技能组***
1. **创建接入网站与技能组设置**  
  + 菜单 客服接入->网站列表，**创建新网站** ***(网站只是一个区分不同接入方的概念，不用填具体网站)***  
  + 点击网站列表中的**接入**按钮，去修改各自的接入配置  
   主要修改技能组：
     + 点击接入，会打开新页面  
     + 点击客服信息，会打开新页面  
     + 找到 9、启用技能组模式，勾选“启用”，勾选“绑定单一技能组”，在“a、
    技能组”下拉选项中，选择刚才创建的机构  
1. 菜单 用户和组->用户账号，创建新用户，按页面输入用户信息，**一定要勾选多媒体坐席**，提交保存  
1. 菜单 用户和组->系统角色，如果没有则 新建角色（普通客服角色），普通角色授权：只需要**坐席对话**即可  
通过**添加用户到角色**按钮，将之前创建的角色添加到该角色中  

#### 新增一个连锁药店后需要做哪些操作
*新增一个连锁药店，按照现有的设计相当于要增加一个 接入网站和技能组，对应的也要创建一个组织机构和坐席用户*  
1. 在 [新增一个接入网站需要做哪些操作](#forJavaDeveloper_manual_addWebSite) 的基础上执行下面的步骤
1. **将客服配置增量初始化到egodrug库**  
  + 在上面第一步时，可以先不创建坐席用户，待存储过程执行完再创建坐席用户，并关联到**角色**和**组织机构**中
  + 在‘客服系统数据库 **cosinee**’ 执行存储过程`call cosinee.pro_init_customer_chat_rel();`增量初始化连锁药店配置到表`egodrug.sh_chain_drugstore_chat_config_relation`  

#### 客服设置相关
管理员账号登录，可以看到左侧菜单栏有一个菜单“客服设置”  
可以修改 对话设置等内容


### <span id="forJavaDeveloper_customerApiOrder">访客端请求主要逻辑&接口顺序</span>
* 以域名 kf.haoyd.com 为例

1. /im/text/{appid}.html  
appid表示接入网站标识，与技能组关联，与组织机构关联，影响坐席分配  
会跳转到 **/apps/im/text.html** 页面，该页面主要是处理上一个请求没有没有传入**userId**用户ID时，会根据浏览器指纹计算生成一个**userId**  
e.g. http://kf.haoyd.com/im/text/154wmf.html  

1. 在text.html页面上发起请求 /im/index.html  
该请求后端主要处理逻辑：  
判断该接入网站 **appid**，是否有效？是否有在线坐席？  
然后根据请求来源跳转 **网页web端页面 index.html**，**或 移动端页面mobile.html**  
e.g. http://kf.haoyd.com/im/index.html?appid=154wmf&orgi=cskefu&client=ce85b1cc61ac4530909c5cf87a0f6b7a&type=text&skill=2c9280866f840b0c016f84125d79006d&userid=b968af729ef7a540e877d14c56f7e3fb&sessionid=fee1fc61-b2f1-4119-ae37-e33a7e30d668&t=1579057249168

1. 访客端通过 websocket连接请求 ws://kf.haoyd.com/socket.io/  
主要逻辑：分配坐席，发送  
e.g. ws://kf.haoyd.com/socket.io/?userid=b968af729ef7a540e877d14c56f7e3fb&orgi=cskefu&session=fee1fc61-b2f1-4119-ae37-e33a7e30d668&appid=154wmf&osname=&browser=&skill=2c9280866f840b0c016f84125d79006d&nickname=Guest_%4019fwrt&isInvite=&EIO=3&transport=websocket

### <span id="forJavaDeveloper_guestInfoExtend">扩展访客信息</span>
> 要扩展访客信息，需要做以下修改

1. cosinee数据库表cosinee.uk_agentuser，增加字段，并增加DTO的字段
1. **第一次接口修改**
   + 接口 /im/text/{appid}（按接入标识分配坐席）和 /im/textbyarea（按地区分配坐席）
     + 入参对象 **OuterUserInfo** 中增加字段
     + 通过 **ModelAndView#addObject(String, Object)** 将字段值传递给下一个页面，可参考
      ```javascript
        地区名称
        view.addObject("areaname", outerUserInfo.getAreaname());
      ```
1. **第一个页面修改**
  + 请求上面的接口后会跳转到页面 **text.html**，需要在该页面再次组装请求参数，在发起请求前组装请求参数，关键代码
   ```javascript
      请求接口 /im/index
      ukefu.openChatDialog();
   ```
1. **第二次接口修改**
  + 上一个页面会重新组装请求参数并请求接口 /im/index，因此
    + 入参对象 **Contacts** 增加扩展字段即可
1. **第二个页面修改**
  + 由于是通过WebSocket将用户信息再传给服务端，才保存，所以需要修改WebSocket连接时参数
  ```javascript
   socket.on('connect',function(){
   <#if contacts?? && contacts.name??>
       	socket.emit('new', {
   			name : "${contacts.name!''}",
   			phone:"${contacts.phone!''}",
   			email:"${contacts.email}",
   			memo:"${contacts.memo!''}",
   			areacode:"${contacts.areacode!''}",
   			areaname:"${contacts.areaname!''}", 按照这种方式添加访客信息
   			orgi:"${inviteData.orgi!''}",
   			appid : "${appid!''}"
   		});
   </#if>
   })
  ```  

### <span id="forJavaDeveloper_db">主要数据库表及关系</span>
**用户与角色**  
uk_role 角色  
cs_user 用户  
uk_userrole 用户角色（哪些用户拥有该角色）  
uk_role_auth 角色授权表（角色有哪些权限）  

**坐席状态**  
uk_work_session 坐席状态  
uk_work_monitor 坐席监控  
uk_webim_monitor 坐席状态  

**组织机构**  
uk_organ 组织 ***（选择开启技能组才能作为技能组关联到渠道(即 接入网站表))***  
cs_organ_user 组织用户关系  

**接入网站设置与技能组**  
uk_snsaccount 渠道表[即 接入网站表]（字段 snsid 为网站接入key）  
uk_consult_invite 接入网站配置  
（有关联技能组的几个字段：skill=1绑定技能组、consult_skill_fixed=1绑定单一技能组、consult_skill_fixed_id=uk_organ.id绑定的单一技能组ID<即 组织机构ID>）  
（关联关系： uk_consult_invite.snsaccountid == uk_snsaccount.snsid）  

**访客与聊天记录**  
uk_agentstatus 在线客服坐席状态表  
uk_agentuser   在线客服访客咨询表（访客与客服对应）userid=snsuser=owner=访客ID，agentno=坐席ID  
uk_agentservice 在线客服服务记录表 agentno=agent=坐席ID  
  （   
	uk_agentuser.id = uk_agentservice.agentuserid 访客ID  
	uk_agentuser.agentserviceid = uk_agentservice.id = uk_agentservice.agentserviceid 服务记录ID  
	uk_agentuser.contextid = uk_agentservice.contextid 会话ID  
	uk_agentuser.sessionid = uk_agentservice.sessionid 会话ID  
   ）  
uk_chat_message 聊天记录  

**系统设置与数据字典**  
uk_systemconfig 系统设置表  
uk_sysdic 字典表  


## <span id="forFrontDeveloper">For前端对接</span>
*无特殊说明则参数类型都为 string 字符串*  

### 根据网站接入标识联系客服
#### 接口地址
https://kf.haoyd.com/im/text/{appid}.html
#### 接口说明
请求该接口后，前端无需再做处理，会打开新页面，客户与坐席对话页面
#### 请求参数

| 参数名       |参数形式| 是否必填| 参数说明     |
|---------     | ----- | :----: | ----------  |
| appid        | path  | **是** | 网站接入标识 |
| userid       | query | 否     | 客户ID（没有传参则会自动生成） |
| name         | query | 否     | 客户名称     |
| phone        | query | 否     | 客户联系方式 |
| description  | query | 否     | 客户首次发送的咨询信息      |
| areacode     | query | 否     | 地区编码（仅作为客户信息保存） |
| areaname     | query | 否     | 地区名称（仅作为客户信息保存） |

#### 响应
跳转到新页面，根据是否有在线坐席可以分配会呈现不同页面  
  + 与坐席聊天页面（有在线坐席可以分配）
  + 留言页面（无在线坐席）

### 根据地区分配客服
#### 接口地址
https://kf.haoyd.com/im/textbyarea.html
#### 接口说明
请求该接口后，前端无需再做处理，会打开新页面，客户与坐席对话页面
#### 请求参数

| 参数名       |参数形式| 是否必填| 参数说明     |
|---------     | ----- | :----: | ----------  |
| userid       | query | 否     | 客户ID（没有传参则会自动生成） |
| name         | query | 否     | 客户名称     |
| phone        | query | 否     | 客户联系方式 |
| description  | query | 否     | 客户首次发送的咨询信息      |
| areacode     | query | **是** | 地区编码（用于按地区分配坐席） |
| areaname     | query | **是** | 地区名称 |

#### 响应
跳转到新页面，若该地区没有对应坐席则提示无匹配坐席  
  + 与坐席聊天页面
