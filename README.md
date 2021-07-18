# agree 已知bug
1 agree大于2.4版本不支持a.b.c语句
2 agree不支持connection时，连接元素组成超过1个".",即a.b可以，但是a.b.c不支持
3 agree xtext不支持结构体直接相等。

# 解决的bug （fixed)
connections连接数,原因是连接时不能超过一个"."

--
# 进度
基本完成，可能存在一定的抽象导致的描述不完整和bug，两个故障分别注入了SB和新加的APP CI、TO中

# Surf_V3
add sth global configuration

The appId for restart need to be different for each core app and common app
	--e.g. core : 0-4, common : > 4
now,
    es:0,
    sb:1, 
    evs:2,
    time:3.
    HS: 5
    CI: 6
    TO: 7

App State:
this is for change the appstate to 1:restart
	--0: running 1: restart, 2:waiting, 4: initialazing

## global MsgId 
for all events, msgid is same, but different evtID
	now set msgId 0 for type event, 
	RestartAPP cmd is set to 0 for evtID
for implementing route table
we should cut msgId into three classes
### MsgId 0: for event
eventType: DEBUG:0x1,INFORMATION:0x2,ERROR:0x4,CRITICAL:0x8,

MsgId 1: for core app's msg 
MsgId 2: for sample app's msg

TO do:
这里假设所有的函数功能，在一次调用后将结果更新至sharedata，即调用和更新发生在一个周期，
tricks：在初始化阶段，屏蔽所有函数调用	

## 现有的核心服务app
四个核心服务的app实际现在不具有实用性，
用于接收地面传来的命令执行对应的操作
现在没有加入遥测、遥控指令

### newList
add maintain function to es
but the guarantee to test is left.
### there seem not to be  a function to let es really restart app
1 each call function inherts from generic ,with call return interface,
That can be used to connect subcomponet function to surf
top model,but haven't been implemented.

2 add a restart surf api to es , for hs
but need to implement
# 安全性机制相关
1 HS的异常事件监听机制——检错
由SB服务提供，可由HS订阅所有的事件，这样HS可监听到异常事件，而需要进行错误报告的事件可自定义，这里先假设了一种错误事件类型

在SB服务中，假设了在初始的时刻，HS订阅事件，且激活管道，并且保持这样的监听状态

SB服务提供了转发事件的机制
### 各种服务调用出错后，会发送错误事件，还未实现，应该通过谁调用出错，谁进行错误报告的机制实现。实际通过调用接口返回值来实现。

2 HS的APP活性监测机制——检错
由ES服务提供，由ES负责更新APP的心跳计数器，并将结果推送至HS

每个APP为了维持自己的活性，需要在每次执行结束后调用runloop激活心跳。需要为每个核心服务新增一个，虽然实际的代码实现不是用的同一个。
appstate：运行状态 
task : execCounter，心跳
应当由appstate控制app的功能执行是否正常，
## HS
HS获取到app信息则，用es中的计数器更新HS中的计数器，且这时调用条件为HS中计数器和ES中计数器不相等，说明HS未能检测到app信息已经过去了几个cycle
获取不到则递减CheckInCounter，若检测余数减少到0，则采取用户预定义的动作，这里可以有重启处理器、重启app、发送错误事件、无动作
HS同时还在主循环中喂看门狗
### 可和SEU引起的组件崩溃联系到一起，通过影响runloop更新，通过appstate对app功能的控制，来体现由于seu导致组件崩溃，功能失效

## EVS_task
### initial
这个初始化的时候订阅了这些消息不知道有啥用
CFE_ES_RegisterApp->EVS_GetAppID->CFE_EVS_Register->CFE_SB_CreatePipe
/* Subscribe to command and telemetry requests coming in on the command pipe */
   Status = CFE_SB_SubscribeEx(CFE_EVS_CMD_MID, CFE_EVS_GlobalData.EVS_CommandPipe,
                               CFE_SB_Default_Qos, CFE_EVS_MSG_LIMIT);
 CFE_SB_SubscribeEx(CFE_EVS_SEND_HK_MID, CFE_EVS_GlobalData.EVS_CommandPipe,
                               CFE_SB_Default_Qos, CFE_EVS_MSG_LIMIT);

### CFE_ES_IncrementTaskCounter
各个核心服务的app调用这个函数直接对计数器增加，其实这个和核心服务库好像没什么关系，

## ES_API
### ES的两个数据结构appRecord和taskRecord
appRecord维护app的运行状态、待执行的控制状态指令
taskRecord维护心跳计数器
### CFE_ES_RunLoop 
会修改appRecord中的状态和taskRecord中的心跳计数器
Runstatus由app自身设置值，一般都为run，初始化失败时会设置为error，这里其实可以先不考虑error的情况，（这里并没有建模初始化一个app的详细过程，或者说把这个初始化过程分摊到多个服务的共享数据的初始化中了，默认都初始化成功了。
Check for Exit, Restart, or Reload commands
本函数主要检查调用者APP的运行状态。通常每个APP在初始化后，会进入主程序循环状态，而每次循环，都会通过CFE_ES_RunLoop来检查当前APP的运行状态。
对该APP的主任务执行计数加一，并根据RunStatus来执行不同的操作。当RunStatus为CFE_ES_RUNSTATUS_APP_RUN时，且AppState为CFE_ES_APP_STATE_INITIALIZING，则将AppState设置为CFE_ES_APP_STATE_RUNNING；
当RunStatus为CFE_ES_RUNSTATUS_APP_EXIT或者CFE_ES_RUNSTATUS_APP_ERROR是，则将RunStatus赋值给AppControlRequest变量；
如果RunStatus不为以上三种类型的话，则将AppControlRequest设置为CFE_ES_RUNSTATUS_APP_ERROR，之后这个APP就会被ES服务所删除。



### CFE_RESTART_APP
 如果APP的类型类型是CFE_ES_APP_TYPE_CORE或者APP当前的状态不为CFE_ES_APP_STATE_RUNNING，则不能重启；否则话则将此APP的AppControlRequest属性设置为CFE_ES_RUNSTATUS_SYS_RESTART，将属性AppState设置为CFE_ES_APP_STATE_WAITING即可。函数最后释放信号量并返回。
 Restart将requestOp设置StateRecord.AppControlRequest为重启,
因为ES服务的主任务中，会定时遍历CFE_ES_Global.AppTable数组去获取所有APP的状态，如果某个APP的AppState属性不为CFE_ES_APP_STATE_RUNNING，根据StateRecord.AppControlRequest执行删除、重启、重新加载操作 。

3 三冗余机制——容错
已知的，已经在surf中完成的有sb中的subscribe，

### 注入单比特翻转seu，影响效果可为使得SB中消息管道中的消息出错，并使用三冗余容错可解决


4 日志信息
重启信息会被保留在日志中，这个并没有添加