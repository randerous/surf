# Surf_V3
add sth global configuration

The appId for restart need to be different for each core app and common app
	--e.g. core : 0-4, common : > 4
now,
    es:0,
    sb:1, 
    evs:2,
    time:3.

App State:
this is for change the appstate to 1:restart
	--0: running 1: restart, 2:waiting
