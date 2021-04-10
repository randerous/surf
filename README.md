# Surf_V3
add sth global configuration

The appId for restart need to be different for each core app and common app
	--e.g. core : 0-4, common : > 4
now,
    es:0,
    sb:1, 
    evs:2,
    time:3.
    sample_app: 5

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
