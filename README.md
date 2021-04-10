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

App State:
this is for change the appstate to 1:restart
	--0: running 1: restart, 2:waiting

MsgId 
for all events, msgid is same, but different evtID
	now set msgId 0 for type event, 
	RestartAPP cmd is set to 0 for evtID
for implementing route table
we should cut msgId into three classes
MsgId 0: for event
MsgId 1: for core app's msg 
MsgId 2: for sample app's msg

TO do:	
each call function inherts from generic ,with call return interface,
That can be used to connect subcomponet function to surf
top model,but haven't been implemented.

# 安全性机制相关
1 HS的异常事件监听机制
由SB服务提供，可由HS订阅所有的事件，这样HS可监听到异常事件，但具体实现不明

SB服务提供了转发事件的机制

2 HS的APP活性监测机制
由ES服务提供，由ES负责更新APP的心跳计数器，并将结果推送至HS


3 三冗余机制
已知的，已经在surf中完成的有sb中的subscribe，