[toc]
# 1.概述
瑞云渲染RenderBus系统除支持网页、客户端操作外，针对大型客户，对外开放了数据接口API，提供高级用户快速创建任务，上传文件、提交渲染，并且对任务进行监控和管理。
# 2. 适用范围
本协议规定了瑞云渲染农场客户端、以及第三方平台与瑞云渲染服务后台之间的技术要求、接口方式、数据格式、传送方式等规范标准。
# 3.术语定义
## 3.1 云渲染平台
瑞云提供的渲染的云平台，可通过网页提交、或客户端工具进行文件上传、渲染和下载。可以帮助用户轻松完成渲染操作，直接调用海量计算资源，强大的管理工具和整合方案。
## 3.2 API接口
云渲染平台提供给第三方直接调用的接口，可实现代码级调用，第三方用户可在API基础上实现大批量文件的快速渲染，并且能够通过API接口，集成到的自建系统。
## 3.3 服务端
云渲染平台提供API接口的服务方，接收任何第三方客户端发送的请求，响应和处理。
## 3.4 客户端
这里可理解为调用方，如接口Http客户端方，第三方系统无论采用自建系统调用、命令行调用、或者浏览器调用，对于API接口，统称客户端。
# 4.相关定义
## 4.1 通讯协议
本接口基于Http/Https作为通信协议，使用Restful作为通讯方式，传输内容基于Json格式。
## 4.2 调用限制
本接口只开放给瑞云授权用户使用，所有接口必须包含授权码ACCESS_KEY（详见章节4.2）
## 4.3 开通方式
瑞云注册用户可联系瑞云服务人员开通API接口调用权限，未注册用户请先注册。瑞云会提供ACCESS_KEY，获取后请妥善保管，注意保密。
## 4.4 快速调用
用户可编写任何语言的Restful客户端代码完成对API的调用。  
除此之外，瑞云渲染平台API提供配套的快速调用工具，提供Python SDK版本的封装工具，你可以使用这些工具快速完成接口调用等，无需研发底层的Restful通讯代码。
# 5.接口调用说明
## 5.1 总体说明
完成云渲染过程通常分为上传场景文件和提交渲染参数及命令两大块。由于场景文件通常较大，传统的接口协议无论是http协议或Tcp协议均无法较好支持大文件的传输，而瑞云本身提供了高速传输工具sync和aspera，所以场景文件的上传，所以目前调用API接口进行任务提交，需先通过高速传输工具完成场景文件的上传。
## 5.2 渲染流程
```
graph TD

A(开始) --> B[创建项目]

B --> C[上传文件到该项目]
C --> D[创建任务]
D --> E[启动任务]
E --> F{渲染文件}
F --> |成功| G[下载渲染文件]
F --> |异常| H[重提任务]
H --> E[启动任务]
G --> I(结束)


```

说明：  
1. V2版接口开放任务的提交、查询和操作以及任务的查询，项目的创建、操作和查询接口，以及插件配置接口，还有用户的查询。
2. 文件的上传和下载可通过sync工具或aspera工具，上传时文件存储的相对路径为后续调用提交任务的关键参数。
## 5.3接口调用地址
### 5.3.1地址样例
接口URL由协议类型、域名、接口版本号、操作对象及ID组成，如下：  
<protocol>://<平台域名>/api/<版本>/<模块>/  
例如:(https://www5.renderbus.com/api/v2/task/)
### 5.3.2平台地址
目前提供2、5平台  
地址前缀：https://www5.renderbus.com/api/v2/task/
## 5.4接口清单

序号 | 类型编码 | 模块| 路径| 提供接口
---|---|---|---|---
1 | 01 | 渲染任务 | row 1 col 4 | 1.提交任务  2.查询任务列表  3.操作任务
2| 02 | 项目相关 | 同上 | 项目查询
# 6.公共协议
## 6.1JSON格式
1. 请求参数必须以json格式提交，并且需设置contentType为application/json，json的最大长度不超过200K字节。  
2. JSON格式由head和body组成，head部分为公共信息部分，所有接口统一，具体规则见章节6.2。body部分则为具体业务规则部分。
3. 非规范中定义的字段不被使用，非必填字段可不填，必填字段则需按规范说明进行填写。
4. 参数或值均区分大小写。
5. 所有字段名均采用全小写，可用下划线分开。
6. 返回结果同样以JSON格式返回。
7. 中文字符长度占2位。
## 6.2head规则
### 6.2.1请求head格式说明
字段 | 中文描述 | 是否必填| 最大长度| 类型| 取值/说明
---|---|---|---|---|---|---
access_key | 身份标识 | Y | 32 | 字符串| | 
account| 调用方账号 | Y | 32 | 字符串| | 
msg_locale| 返回信息语言 | N |  | 字符串| zh/en，暂时不用| 
action| 操作方法 | Y | 32 | 字符串| 取各接口说明| 
### 6.2.2head格式说明
字段 | 中文描述 | 说明|
---|---|---|---|---
result | 执行结果 | 0：成功 1：失败
error_code| 错误码 |执行失败时的错误码，通用错误可参见附录A。
description| 错误信息 | 错误码对应的描述信息。
## 6.3body规则
所有接口body规则详见第7大章业务协议，请求参数规范注明了客户端请求参数的body规则，返回值规范注明了返回结果的body规则。
## 6.4超时时间
本接口协议规定，对于调用方先发起的数据请求，服务方将根据请求类型进行响应，所有请求均需进行响应应答，否则调用方在超过超时时间后将认为数据请求失败。本系统默认超时时间为30秒。
## 6.5加密规则
为了保证数据在传输过程中不被窃取，本接口协议对Json进行加密传输,V1版本暂不实现。
# 7.业务协议
## 7.1任务模块
### 7.1.1创建任务
说明：创建一个任务。  
注意，这里的任务仅适用于Maya任务，其他任务，比如Max和Sketchup，因为参数只存在稍微区别，所以我们将其请求参数放入附录中。任务类型虽不同，但返回值相同。  
action：create_task
#### 7.1.1.1请求参数
字段名 | 中文含义 | 是否必填| 参数类型| 最大长度 | 说明
---|---|---|---|---|---|---
submit_account | 客户编号 | N | 字符串 | 64|提交该任务的账号务账号，缺省时取head里的。 | 
project_name| 项目名称 | Y | 字符串 | 64|如果项目名称匹配不上，将无法提交。 | 
input_scene_path| 场景路径 | Y | 字符串 | 512| 支持多个文件批量提交任务，以半角逗号分隔,最大支持100条，超过100系统会返回错误信息。| 
cg_soft_name| 渲染软件 | Y | 字符串 | 64| | 
frames| 提交帧 | Y | 字符串 | 64|要渲染的帧|
multi_pass| 通道 | N | 整型 |  |0：不需要；1：需要| 
kg| 渲染类型 | N | 字符串 | 64 |Max渲染时必需要填，否则有可能导致渲染失败| 
output_file_name| 输出文件的名称 | N | 字符串 | 64 |为空时，默认为场景名|
output_file_format| 输出文件的格式 | N | 字符串 | 64 |为空时，则按默认格式输出|
image_width| 像素宽 | N | 整型 |||
image_height| 像素高 | N | 整型 |||
attr1| 扩展使用| N |字符串 |64|V1.0版本可不使用|
attr2| 扩展使用| N |字符串 |4000|V1.0版本可不使用|
tiles| 分块渲染| N |字符串 |32|V1.0版本可不使用|
timeout| 分块渲染| N |整型 ||单位秒|
plugin_name| 渲染插件| N |字符串|512|多个插件以逗号分隔|
camera| 渲染相机| N |字符串|64|渲染相机|
render_layer| 渲染层 | N|字符串|64|渲染层|
remark| 备注 | N|字符串||任务备注|
artist| 制作人 | N|字符串||制作人|
submit_type| 提交类型 | N|整型||提交任务后是否直接开始渲染，默认为0 。          0：直接开始渲染 1：将任务设为停止状态|
environments| 环境变量|N|字符串||设置系统的环境变量|
#### 7.1.1.2返回值
字段名 | 中文含义 | 参数类型|  说明
---|---|---|---|---|---|---
data | 数据集合 | 数组集合 | 返回结果的数据集合，详细描述见Data字段说明。|
total_size| 数据总条数 | 字符串 | 返回结果数据集合的总条数。|
**data字段说明**
字段名 | 中文含义 | 参数类型|  说明
---|---|---|---|---|---|---
task_id | 任务id | 字符串| 平台生成的任务id，一个场景文件对应一个任务id。|
submit_result| 提交结果 | 字符串| 0：成功；1：失败。|
#### 7.1.1.3业务规则说明
提交任务业务规则说明如下：  
1. submit_account与head中的account有一定区别，head中的account一般使用主账号，代表使用接口的账号，submit_account则为提交该任务时的账号，可以是主账号也可以是子账号。
2. 提交任务前务必保证待渲染的场景文件已经上传，且input_scene_path填写正确，否则将会渲染失败。请详细阅读章节5.1接口渲染流程。
3.  frames的具体规则：  
      1. 单帧：格式如“1,2,3,4,5,8,12”，中间以半角逗号分隔。
      2. 序列帧: 格式如“1-20或1-20[2]”，前面1-20代表帧的范围，后面的[2]代表步长，非必填，默认为1。隔帧举例：如序列帧格式为1-20[2]，实际渲染帧为“1,3,5,7,9......”
      3. 复杂帧：复杂帧为单帧模式与序列帧模式的组合，格式如“1,20-30[2],494”。
4.  一次提交多个场景文件时，系统会自动生成多个任务，返回结果以列表形式返回。单个场景文件提交时也会返回列表，列表的条数为1。
5.  一机渲多帧机制需要客户在平台客户配置中预先配置好。
6.  iformat格式如有设置，需与渲染器匹配，否则会导致渲染失败。
#### 7.1.1.4JSON样例
```python
{
  "head": {
    "access_key": "0WEQE123FDASDF",
    "account": "012345678901234567890123456789",
    "msg_locale": "zh",
    "action": "create_task"
  },
 "body": {
    "submit_account": "frank01",
    "project_name":"test",
    "input_scene_path":"D/maya/zulu.mb",
    "cg_soft_name":"maya 2015",
    "frames":"1-10"
  }
}
```
### 7.1.2查询任务
字段名 | 中文含义 | 是否必填| 参数类型| 最大长度 | 说明
---|---|---|---|---|---|---
task_id | 任务ID | N | 整型 ||精确查询 | 
project_name | 项目名称 | N | 字符串 |32|精确查询 | 
is_sub_included | 是否包含子账号 | N | 整型 ||0：不包含（默认值）；1：包含| 
is_jobs_included| 是否包含任务详情 | N | 整型 ||0：不包含（默认值）；1：包含| 
is_logs_included| 是否显示渲染帧的详细日志 | N | 字符串 ||需要is_jobs_included为1时才有效。0：不显示（默认值）；1：显示渲染日志| 
task_status| 任务状态 | N |字符串||"Start"|"SubFailed"|"Stopped"|"System_Done"|"Failed"|"User_Stop"|"executing"|"Preprocessing"|"Analysed"|"Analysing"|"AnalyseFailed"| 
del_flag| 任务是否删除 | N | 整型 ||0未删除（默认值）；1查询删除任务| 
task_type| 渲染软件 | N | 整型 ||0：主图；1：光子；2：光子+主图| 
cg_soft_name| 任务是否删除 | N |字符串 |32|模糊查询| 
input_scene_path| 场景名 | N | 字符串 |256|模糊查询| 
camera| 渲染相机 | N | 字符串 |256|模糊查询| render_layer| 渲染层 | N | 字符串 |256|模糊查询|
start_date_after| 开始时间大于等于 | N | 字符串 |32|格式：yyyy-MM-dd HH:mm:ss；  实例：2016-08-31 23:59:59|
start_date_before| 开始时间小于等于 | N | 字符串 |32|格式：yyyy-MM-dd HH:mm:ss；实例：2016-08-31 23:59:59|
completed_date_after| 任务完成时间大于等于 | N | 字符串 |32|格式：yyyy-MM-dd HH:mm:ss；实例：2016-08-31 23:59:59。使用该参数时请指定任务状态为System_Done，否则不保证结果的正确性|
completed_date_before| 任务完成时间小于等于 | N | 字符串 |32|格式：yyyy-MM-dd HH:mm:ss；实例：2016-08-31 23:59:59。使用该参数时请指定任务状态为System_Done，否则不保证结果的正确性| 
task_over_time| 超时时间 | N | 字符串 ||任务的超时时间| 
submit_account| 提交账号 | N | 字符串 |32|提交该任务的实际账号| 
artist| 制作人 | N | 字符串 |32|模糊查询| 
sort_code| 排序码 | N | 整型 ||0：任务ID排序；1：任务状态排序；2：任务提交时间排序| 
sort_type| 排序方式 | N | 整型 ||0：顺序（默认值）；1：倒序只有当sort_code不为空时，该字段才生效。| 
is_paging| 是否分页 | N | 整型 ||0：不分页（默认值）；1：分页。| 
page_size| 分页-每页条数 | N | 整型 ||当is_paging为1时，该字段生效，且为必填字段，如果为空，则默认100。| 
page_no| 分页-第几页 | N | 整型 ||当is_paging为1时，该字段生效，且为必填字段，如果为空则默认为1。| 
#### 7.1.2.2返回值
说明：只有当用户选择分页时，is_paging，page_no，page_size才会显示，同理，当用户不使用排序时，sort_code，sort_type是不可见的。
字段名 | 中文含义 | 数据类型|  说明
---|---|---|---|---|---|---
data| 数据集合| 数组集合 | 返回结果的数据集合，详细描述见Data字段说明。查无记录时该字段为空。 |
total_size | 数据总条数 | 整型 | 返回结果数据集合的总条数。 |
page_no | 当前页码 | 整型 | 返回查询的当前页码。 |
total_page | 总页数 | 整型 | 返回查询总页数。 |
page_size| 每页条数| 字符串| 返回查询时的每页条数。 |
sort_code| 排序码| 字符串| 返回查询时的排序码。 |
sort_type| 排序方式| 字符串| 返回查询时的排序方式。 |
**data字段说明**
字段名 | 中文含义 | 数据类型|  说明
---|---|---|---|---|---|---
task_id| 任务id| 字符串 | 任务id。 |
project_name | 项目名称 | 字符串 | 项目名称。 |
task_status | 任务状态| 字符串 | 任务状态。|
frames| 渲染帧| 字符串 | 格式：1-10[1]。|
task_type| 任务类型| 字符串 | 0：主图;1：光子;2：光子+主图|
frames| 渲染帧| 字符串 | 格式：1-10[1]。|
done| 成功帧数| 字符串 |已经渲染成功的帧数图|
failed| 失败帧数| 字符串 |正在执行渲染的帧数|
failed| 失败帧数| 字符串 |渲染失败的帧数|
executing| 执行帧数| 字符串 |正在执行渲染的帧数|
waiting| 等待帧数| 字符串 |等待渲染的帧数|
aborted| 放弃帧| 字符串 |已放弃帧数|
cg_soft_name| 渲染软件| 字符串 |渲染软件|
input_scene_path| 场景名| 字符串 |场景文件的名称|
fee| 任务费用| 字符串 |该任务已经产生的费用，单位元|
submit_time| 开始时间| 字符串 |格式：yyyy-MM-dd HH:mm:ss|
completed_time| 任务完成时间| 字符串 |格式：yyyy-MM-dd HH:mm:ss|
exetime_count| 所有帧的总时间| 字符串 |所有帧的总时间|
camera| 渲染相机| 字符串 |渲染相机|
render_layer| 渲染层| 字符串 |渲染层|
artist| 制作人| 字符串 |制作人|
submit_account| 提交账号| 字符串 |提交该任务的实际账号|
jobs| 任务详情| JSON数组 |只有当is_jobs_included为1时，此数据才会显示。job_id：帧ID; job_name：帧名称;job_status：帧状态;start_time：开始时间，格式yyyyMMddHHmmss;end_time：结束时间，格式yyyyMMddHHmmss;consume_time：共耗时，单位s;log：渲染日志|
#### 7.1.2.3业务规则说明
查询任务业务规则说明如下：
1. 只能查询head参数中account用户（或其子账户）下的任务。
2. 查询单条信息时，使用task_id为参数精确查询即可。
3. 任务查询服务只支持主动查询，不支持任务状态推送功能。
#### 7.1.2.4JSON样例
```python
{
    "body": {
        "data": [{
            	"task_id": "5101184",
            	"project_name": "J_Team",
            	"task_status": "Start",
        	    "task_type": "0",
            	"cg_soft_name": "maya2014",
            	"scene_name": "Ep045_SC008__v001.ma",
            	"frames": "101-123[1]",
            	"executing": "3",
            	"done": "7",
            	"aborted": "0",
            	"waiting": "13",
            	"failed": "0",
            	"submit_time": "2016-10-11 14:26:03",
            	"fee": "RMB:2.58",
            	"jobs": [
                	{
                    	"job_id": "5",
                        "job_name": "frame0106",
                    	"job_status": "Done",
                    	"start_time": "20161011144433",
                    	"end_time": "20161011145128",
                        "consume_time": "415"，
            "log": "abc\r\ndef\r\nhij\r\n"，
                        "render_log": "abc\r\ndef\r\nhij\r\n"，
"process_log": "abc\r\ndef\r\nhij\r\n"

                    },
                	{
                    	"job_id": "6",
                    	"job_name": "frame0107",
                    	"job_status": "Done",
                    	"start_time": "20161011144439",
                  	 "end_time": "20161011145024",
                        "consume_time": "345"，
 	"log": "abc\r\ndef\r\nhij\r\n"，
                        "render_log": "abc\r\ndef\r\nhij\r\n"，
"process_log": "abc\r\ndef\r\nhij\r\n"
                	},
                               	...
            	]
        	}
        ],
        "total_size": 1
    },
    "head": {
        "result": "0"
    }
}

```
### 7.1.3任务操作（Python接口暂未实现）
action：operate_task
#### 7.1.3.1请求参数
字段名| 中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|---
task_id| 任务ID|Y|字符串|256|支持批量操作，批量时id用半角逗号隔开|
operate_order| 操作指令|Y|整型| |0：暂停任务；1：重提任务；2：删除任务；3：修改任务参数|
restart_type| 重提类型|N|整型| |0：重提失败帧；1：重提放弃帧；2：重提完成帧；3：重提开始帧；4：重提等待帧；当operate_order 为1时，该字段需填值。否则默认为0|
task_over_time| 超时时间|N|整型| |operate_order为3时会修改超时时间。|
#### 7.1.3.2返回值
字段名| 中文含义|数据类型|说明|
---|---|---|---|---
data| 数据集合|数组|返回结果的数据集合，详细描述见Data字段说明|
total_size| 数据条数|字符串|返回结果数据集合的总条数|
**data字段说明**
字段名| 中文含义|数据类型|说明|
---|---|---|---|---
task_id| 任务ID|字符串|请求参数中的task_id|
operate_result| 操作结果|字符串|0:成功；1:失败|
task_status| 任务状态|字符串|任务状态|
#### 7.1.3.3业务规则说明
任务操作有一定的业务规则限制，说明如下：
1. 只允许操作本账号下的任务，如果任务id与账号不匹配，系统将返回错误信息。
2. 只有运行状态的任务才能执行停止操作，否则系统会给出错误提示。
3. 任何状态均可执行重提操作，并且需按规则提交重提类型和帧数参数，否则系统会给出错误提示。
4. 只有停止状态的任务才能执行删除操作，否则系统会给出错误提示。
5. task_id查无记录时会提示错误信息。
#### 7.1.3.4JSON样例
**请求参数**
```python
{
  "head": {
	"access_key": "03mnjfytdskjfsa3bhf=9lopdhe",
	"account": "fivemine",
	"msg_locale": "zh",
	"submit_date": "2016-08-25 20:08:12",
	"action": "operate_task"
  },
 "body": {
	"task_id": "900001",
	"operate_order": "restart",
	"restart_type": 0,
	"frame": "1-100[1]"
  }
}

```
**返回值**
```python
{
  "head": {
    "result": "03mnjfytdskjfsa3bhf=9lopdhe",
    "error_code": "fivemine",
    "message": "zh"
  },
 "body": {
    "total_size": "1",
    "data": [ {"list_no":1,
         "task_id":"900001",
         "operate_result":0,
         "result_msg":"",
         "status":3
        } ]
  }
}

```
## 7.2项目模块
### 7.2.1查询项目
说明：注意cg_soft_name、plugin_name与is_default这三个参数，因为项目与软件是一对多（多个软件多条记录），软件与插件也是一对多（多个插件以逗号分隔），所以在使用这三个参数查询时，即使两次查询条件完全对立，也可能查出同一个项目。      action：query_project
#### 7.2.1.1请求参数
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|---|
project_id|项目ID|N|整型||精确查询|
project_name|项目名称|N|字符串|32|精确查询|
project_path|项目路径|N|字符串|256|精确查询|
cg_soft_name|渲染软件|N|字符串|32|模糊查询|
plugin_name|渲染插件|N|字符串|32|精确查询，即插件名称要正确，否则将得不到结果|
plugin_name|渲染插件|N|字符串|32|精确查询，即插件名称要正确，否则将得不到结果|
render_os|渲染系统|N|字符串||Windows|Linux|
is_default|是否是默认配置|N|整型||0：否（默认值）;1：是|
remark|项目备注|N|字符串|256|模糊查询|
sub_account|子账户|N|字符串|32|查询该子账户下的项目|
is_sub_included|是否包含子账户|N|整型||0：否（默认值）;1：是|
#### 7.2.1.2返回值
字段名|中文含义|数据类型|说明|
---|---|---|---|
data|数据集合|数组集合|返回结果的数据集合，详细描述见Data字段说明。|
total_size|数据总条数|字符串|返回结果数据集合的总条数。|
**data字段说明**
字段名|中文含义|数据类型|说明|
---|---|---|---|
project_id|项目id|字符串|项目id|
project_name|项目名称|字符串|项目名称|
project_path|工程路径|字符串|工程路径|
render_os|渲染系统|字符串|渲染系统|
remark|项目备注|字符串|项目备注|
customer_name|所属客户|字符串|所属客户|
create_time|创建时间|字符串|创建时间|
plugins|配置相关|JSON数组|config_id : 配置ID；cg_soft_name：渲染软件；plugin_name：渲染插件，如果某一软件下有多个插件，则以逗号分隔；is_default：该配置是否为默认配置。|
### 7.2.2查询插件
action：query_plugin
#### 7.2.2.1请求参数
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
cg_soft_name|渲染软件|N|字符串|32|模糊查询|
plugin_name|渲染插件|N|字符串|32|精确查询，即插件名称要正确，否则将得不到结果|
#### 7.2.2.2返回值
字段名|中文含义|数据类型|说明|
---|---|---|---|
cg_soft_name|渲染软件|字符串|渲染软件|
plugin_name|渲染插件|字符串|渲染插件|
### 7.2.3创建项目
action：create_project
#### 7.2.3.1请求参数
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
project_name|项目名称|Y|字符串|20|数字或英文字母（不区分大小写）|
project_path|项目路径|N|字符串|256|项目路径|
render_os|渲染系统|N|字符串||Windows|Linux|
remark|项目备注|N|字符串|256|备注|
sub_account|子账户|N|字符串|32|该项目的所有者，如果为空，则取head里的account|
#### 7.2.3.2返回值
字段名|中文含义|数据类型|说明|
---|---|---|---|
project_id|项目id|字符串|项目id|
create_result|创建项目结果|字符串|0：成功;1：失败（这里的失败不包含验证失败）|
description|失败详情|字符串||
### 7.2.4操作项目
action：operate_project
#### 7.2.4.1请求参数
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
project_id|项目id|Y|整型||即将要操作的项目，包括修改项目信息及添加或更改配置。|
sub_account|子账户|N|字符串|32|所操作项目所属账户，默认为head里的account。|
project_name|项目名称|N|字符串|20|数字或英文字母（不区分大小写）。|
project_path|项目路径|N|字符串|256|项目路径|
render_os|渲染系统|N|字符串||Windows|Linux|
remark|项目备注|N|字符串|256|备注|
operate_type|配置操作类型|N|||0：新增；1：修改；2：删除
如果该字段为空，则下面三个参数将被忽略。|
config_id|配置ID|N|整型||新增操作，该字段如果存在，将被忽略；修改操作，如果该字段为空，则返回失败；删除操作，如果该字段为空，则删除所有。|
cg_soft_name|渲染软件|N|字符串|32|新增和修改操作，渲染软件不能为空，否则返回错误。|
plugin_name|渲染插件|N|字符串|256|渲染插件，多个插件以逗号分隔，同一名字不同版本的多个插件只能出现一个，如果出现多个，则返回错误。|
is_default|是否为默认配置|N|整型||0：否；1：是；同一项目下仅允许有一个默认配置。|
#### 7.2.4.2返回值
字段名|中文含义|参数类型|说明|
---|---|---|---|
operate_result|创建项目结果|字符串|0：成功；1：失败（这里的失败不包含验证失败）。|
description|失败详情|字符串||
## 7.3用户模块
### 7.3.1查询用户
action：query_customer
#### 7.3.1.1请求参数
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
login_name|用户名|N|字符串||如果该字段为空，且head中的账号为主账户，则查询该主账户及其所有子账户；否则查询此用户名的用户。|
#### 7.3.1.2返回值
字段名|中文含义|数据类型|说明|
---|---|---|---|
data|数据集合|数组集合|返回结果的数据集合，详细描述见Data字段说明。|
total_size|数据总条数|字符串|返回结果数据集合的总条数。|
**data字段说明**
字段名|中文含义|数据类型|说明|
---|---|---|---|
id|用户id|字符串|该用户id|
login_name|用户名|字符串|该用户登录名称|
is_student|是否为学生|字符串|0：否；1：是|
job|职位|字符串|该用户职位|
role|VIP等级|字符串|49: VIP0, 51/55: VIP1, 56: VIP2，57: VIP3, 58: VIP4|
email|用户邮箱|字符串|该用户邮箱|
phone|手机号|字符串|该用户手机号|
skype||字符串||
ip|ip地址|字符串||
source_url|来源url|字符串|指注册时的来源url|
country|国家|字符串|该用户所在国家|
city|城市|字符串|该用户所在城市|
address|住址|字符串|具体住址|
company|公司名|字符串|公司名称|
register_date|注册日期|字符串|注册日期|
login_date|登录日期|字符串|指最近一次登录日期|
rmb_balance|RMB余额|字符串||
usd_balance|USD余额|字符串||
eur_balance|EUR余额|字符串||
gbp_balance|GBP余额|字符串||
account_type|账户类型|字符串|1：主账户；2：子账户|
zone|国内外|字符串|1：国内；2：国外|
area_id|区域|字符串||
### 7.3.2修改用户
action：update_customer
#### 7.3.2.1请求参数
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
login_name|用户名|N|字符串||如果该字段为空，且head中的账号为主账户，则查询该主账户及其所有子账户；否则查询此用户名的用户。|
notification_type|通知类型|N|整型||任务事件通知，0：不通知（默认）；1：任务开始通知；2：任务完成通知；3：任务失败通知|
notification_mode|通知方式|N|整型||notification_type不为时0生效，1：接口RUL；2：邮箱；3：短信|
notification_address|通知地址|N|字符串||对应notification_node填写URL、邮箱或手机号码|
#### 7.3.2.2返回值
字段名|中文含义|数据类型|说明|
---|---|---|---|
data|数据集合|数组集合|返回结果的数据集合，详细描述见Data字段说明。|
total_size|数据总条数|字符串|返回结果数据集合的总条数。|
**data字段说明**
字段名|中文含义|数据类型|说明|
---|---|---|---|
login_name|用户名|字符串|用户名|
operate_result|操作结果|字符串|0:成功；1:失败|
description|结果说明|字符串|如果失败的话，会有对于的结果说明|
# 8.附录A 通用错误码
错误码|错误信息|
---|---|
1001|Json empty content|
1002|json解析异常|
1004|不支持的Content-Type|
1005|json超过最大长度|
1006|Json format error, head lost|
1007|Json format error, body lost|
1010|access_key be disabled|
1012|account not exist|
1013|account be disabled|
1020|null not permitted；注：未设置或空字符串|
1021|parameter cast exception；注：用户请求字段均为字符串，而有时我们要转成整型或长整型来用|
1022|illegal value；注：有些字段只能取特定值，之外均为非法值|
1023|parameter length exceed scope|
1024|illegal characters be contained|	
# 9.附录B Python SDK说明
https://github.com/renderbus/python-api
# 10.Max任务、Sketchup与VRStandalone任务提交参数
## 10.1Max任务提交参数
action：create_max_task
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
submit_account|客户编号|N|字符串|64|提交该任务的账号务账号，缺省时取head里的。|
project_name|项目名称|Y|字符串|64|如果项目名称匹配不上，将无法提交。|
input_scene_path|场景路径|Y|字符串|512|支持多个文件批量提交任务，以半角逗号分隔,最大支持100条，超过100系统会返回错误信息。|
cg_soft_name|渲染软件|Y|字符串|64||
frames|提交帧|N|字符串|64|要渲染的帧|
output_file_name|输出文件的名称|N|字符串|64|为空时，默认为场景名。|
output_file_format|输出文件格式|N|字符串|64|为空时，则按默认格式输出。|
image_width|像素宽|N|整型|||
image_height|像素高|N|整型|||
tiles|分块渲染|N|字符串|32|V1.0版本可不使用|
timeout|超时时间|N|整型||单位秒|
camera|渲染相机|N|字符串|64|渲染相机|
environments|环境变量|N|字符串||设置系统的环境变量|
## 10.2Sketchup任务提交参数
action：create_sketchup_task
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
submit_account|客户编号|N|字符串|64|提交该任务的账号务账号，缺省时取head里的。|
project_name|项目名称|Y|字符串|64|如果项目名称匹配不上，将无法提交。|
input_scene_path|场景路径|Y|字符串|64|如果项目名称匹配不上，将无法提交。|
input_scene_path|场景路径|Y|字符串|512|支持多个文件批量提交任务，以半角逗号分隔,最大支持100条，超过100系统会返回错误信息。|
cg_soft_name|渲染软件|Y|字符串|64||
output_file_format|输出文件格式|N|字符串|64|为空时，则按默认格式输出。|
image_width|像素宽|Y|整型|||
image_height|像素高|Y|整型|||
plugin_name|渲染插件|N|字符串|512|多个插件以逗号分隔。|
use_vary|是否使用Vary|Y|整型||0 false，1 true（默认）|
page_name|标签|Y|字符串|64|场景中标签名称。|
render_type|渲染类型|Y|整型|64|1表示动画渲染；2表示效果图渲染。|
total_frame|总帧数|N|整型||仅适用于动画渲染。|
total_time|	总时长|N|整型||仅适用于动画渲染。|
environments|环境变量|N|字符串|||
## 10.3 VRStandalone任务提交参数
action：create_vrstandalone_task
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
submit_account|客户编号|N|字符串|64|提交该任务的账号务账号，缺省时取head里的。|
project_name|项目名称|Y|字符串|64|如果项目名称匹配不上，将无法提交。|
input_scene_path|场景路径|Y|字符串|512|支持多个文件批量提交任务，以半角逗号分隔,最大支持100条，超过100系统会返回错误信息。|
cg_soft_name|渲染软件|Y|字符串|64|渲染软件。|
frames|提交帧|Y|字符串|64|要渲染的帧。|
multi_pass|通道|N|整型||0：不需要；1：需要
|
output_file_name|输出文件的名称|N|字符串|64|为空时，默认为场景名。|
output_file_format|输出文件格式|N|字符串|64|为空时，则按默认格式输出。|
image_width|像素宽|N|整型||像素宽。|
image_height|像素高|N|整型||像素高。|
plugin_name|渲染插件|N|字符串|512|多个插件以逗号分隔。|
first_frames|优先渲染帧|N|字符串||平台将优先渲染并最多提供三台机器渲染优先帧，用逗号隔开,例如1,3,5必须与下面的参数结合才有效。|
after_first_render|优先渲染类型|N|字符串||fullSpeed：优先渲染后，自动全速渲染；stop：优先渲染后，停止任务；不填则不进行优先渲染，直接正式渲染。|
environments|环境变量|N|字符串||设置系统的环境变量。|
limitHosts|分部署机器数量|N|字符串||大于1：进行分布式渲染，机器数量为给的值
不填或小于等于1：不进行分布式渲染。|
### 10.3.1注意事项
VRStandalone只支持一帧一个文件，如果一个场景文件中含有多帧，请重新生成。
### 10.3.2VRStandalone请求示例
```python
{
  "head": {
    "access_key": "MjAxNzA3MjExMjE3MDA0MDQwNjcwMDQ=",
    "account": "wupeng",
    "msg_locale": "zh",
    "action": "create_vrstandalone_task"
  },
 "body": {
    "project_name":"defaultProject",
    "input_scene_path":"D/maya/vr_test1.vrscene",
    "cg_soft_name":"V-Ray_standalone",
     "plugin_name":"vray 3.10.01",
    "frames":"1",
    "image_width":"320",
    "image_height":"640"
  }
}
```
**多文件一次渲染写法：**
"input_scene_path":"D/maya/vr_tst_####.vrscene 1-24",  
 这里是上传了24个文件、命名####是从0001-0024这样子，此写法可以直接一次将24个文件都渲染，对应的"frames":"1-24"可以指定渲染哪些帧。    
 **普通单文件写法**
 如果只有单独一个文件可以这样写
 ```python
"input_scene_path":"D/maya/vr_tst_0001.vrscene",
"frames":"1"
```
## 10.4 houdini任务提交参数
action：create_houdini_task
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
project_name|项目名称|Y|字符串|64|如果项目名称匹配不上，将无法提交。|
input_scene_path|场景路径|Y|字符串|512|场景文件路径，必须提前上传。|
cg_soft_name|渲染软件|Y|字符串|64|渲染软件。|
layer_list|提交的节点帧集合|Y|字符串|64|包含三个参数，frames:节点帧数；layerName:节点名；option:-1代表渲染，其余数值代表解算，具体使用方法请看下面示例。|
plugin_name|渲染插件|N|字符串|512|多个插件以逗号分隔。| 
environments|环境变量|N|字符串|512|设置系统的环境变量。| 
### 10.4.1houdini请求示例
layer_list是个json数组，里可以包含多个节点
 ```python
{
  "head": {
    "access_key": "MjAxNzA3MjExMjE3MDA0MDQwNjcwMDQ=",
    "account": "wupeng",
    "msg_locale": "zh",
    "action": "create_houdini_task"
  },
 "body": {
    "project_name":"defaultProject",
    "input_scene_path":"D/renderFile/test.hip",
    "cg_soft_name":"houdini 13",
    "layer_list":  [
               {
               "frames":"1-240[1]",
               "layerName":"/obj/grid_object1/rop_geometry1",
               "option":"0"
               },
               {
               "frames":"1-20[1]",
               "layerName":"/out/mantra1",
               "option":"-1"
               }
     ]
  }
}
```
## 10.5Blender任务提交参数
action：create_blender_task
字段名|中文含义|是否必填|参数类型|最大长度|说明|
---|---|---|---|---|---|
submit_account|客户编号|N|字符串|64|提交该任务的账号务账号，缺省时取head里的。|
project_name|项目名称|Y|字符串|64|如果项目名称匹配不上，将无法提交。|
input_scene_path|场景路径|Y|字符串|512|场景文件路径，必须提前上传。|
cg_soft_name|渲染软件|Y|字符串|64|渲染软件。|
frames|提交帧|Y|字符串|64|要渲染的帧。|
plugin_name|渲染插件|N|字符串|512|多个插件以逗号分隔。|
### 10.5.1
Blender请求示例
 ```python
{
  "head": {
	"access_key": "MjAxNzA3MjExMjE3MDA0MDQwNjcwMDQ=",
	"account": "wupeng",
	"msg_locale": "zh",
	"action": "create_blender_task"
  },
 "body": {
    	"project_name":"defaultProject",
    	"input_scene_path":"D/renderFile/blender276_test/test_276.blend",
	"cg_soft_name":"blender 2.78c",
     	"frames":"1"
  }
}

```



