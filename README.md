# mall

# 易方科技客服系统
目录  
[For后端开发人员](#forJavaDeveloper)  
[For前端对接](#forFrontDeveloper)  

**关于该客服系统**
该客服系统基于第三方开源客服系统（春松客服）做修改，未采用前、后端分离

## 关于春松客服的说明
**开源项目地址：** [https://github.com/chatopera/cosin](https://github.com/chatopera/cosin)

**开发环境搭建：** [https://github.com/chatopera/cosin/wiki/春松客服：开发环境](https://github.com/chatopera/cosin/wiki/%E6%98%A5%E6%9D%BE%E5%AE%A2%E6%9C%8D%EF%BC%9A%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)

**基于Docker的部署方案：** [https://github.com/chatopera/cosin/wiki/服务器部署](https://github.com/chatopera/cosin/wiki/%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%A8%E7%BD%B2)

### 普通部署方案
#### 依赖中间件
* redis:5.0.5  
* mysql:5.7.27  
* activemq:5.14.3  
* elasticsearch:2.4.6

#### 部署方式&步骤
1. 默认端口 **8035**，WebSocket通信端口 **8036**
1. 因为有内嵌 Tomcat可以直接java -jar 启动war包
1. maven打war包 ``mvn clean install -Dmaven.test.skip=true ``
1. 可通过 ``nohup java -jar kefu.war & `` 启动部署项目，这种方式则日志全部输出到 war包同目录的文件nohup.out中，若想将日志分开则不要nohup
1. 是否启动成功可以通过 **nohup.out** 日志文件查看，启动大概需要40秒

## <span id="forJavaDeveloper">For后端开发人员</span>

**后端向前端页面传参方式**  
ModelAndView  
ModelMap  

**html页面**  
页面大量使用了FreeMarker模板语言来处理参数和页面动态展示，若要改页面则需要对FreeMarker有一定了解

### 用户端请求接口顺序
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

1. websocket连接请求 ws://kf.haoyd.com/socket.io/  
主要逻辑：分配坐席，发送  
e.g. ws://kf.haoyd.com/socket.io/?userid=b968af729ef7a540e877d14c56f7e3fb&orgi=cskefu&session=fee1fc61-b2f1-4119-ae37-e33a7e30d668&appid=154wmf&osname=&browser=&skill=2c9280866f840b0c016f84125d79006d&nickname=Guest_%4019fwrt&isInvite=&EIO=3&transport=websocket

### <span id="forFrontDeveloper">For前端对接</span>
*无特殊说明则参数都为 string 字符串*  

#### 根据网站接入标识联系客服
##### 接口地址
http://kf.haoyd.com/im/text/{appid}.html
##### 接口说明
请求该接口后，前端无需再做处理，会打开新页面，客户与坐席对话页面
##### 请求参数

| 参数名       |参数形式| 是否必填| 参数说明     |
|---------     | ----- | :----: | ----------  |
| appid        | path  | **是** | 网站接入标识 |
| userid       | query | 否     | 客户ID（没有传参则会自动生成） |
| name         | query | 否     | 客户名称     |
| phone        | query | 否     | 客户联系方式 |
| description  | query | 否     | 客户首次发送的咨询信息      |
| areacode     | query | 否     | 地区编码（仅作为客户信息保存） |
| areaname     | query | 否     | 地区名称（仅作为客户信息保存） |

##### 响应
跳转到新页面，根据是否有在线坐席可以分配会呈现不同页面  
+ 与坐席聊天页面（有在线坐席可以分配）
+ 留言页面（无在线坐席）

#### 根据地区分配客服
##### 接口地址
http://kf.haoyd.com/im/textbyarea.html
##### 接口说明
请求该接口后，前端无需再做处理，会打开新页面，客户与坐席对话页面
##### 请求参数

| 参数名       |参数形式| 是否必填| 参数说明     |
|---------     | ----- | :----: | ----------  |
| userid       | query | 否     | 客户ID（没有传参则会自动生成） |
| name         | query | 否     | 客户名称     |
| phone        | query | 否     | 客户联系方式 |
| description  | query | 否     | 客户首次发送的咨询信息      |
| areacode     | query | **是** | 地区编码（用于按地区分配坐席） |
| areaname     | query | 否     | 地区名称 |

##### 响应
跳转到新页面，若该地区没有对应坐席则提示无匹配坐席  
+ 与坐席聊天页面
