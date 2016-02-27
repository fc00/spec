# Introduction

Pipe.c:incoming
TUNAdapter.c:incomingFromTunIf
UpperDistributor.c:incomingFromTunAdapterIf
SessionManager.c:incomingFromInsideIf
SessionManager.c:readyToSend
SwitchAdapter.c:incomingFromSessionManagerIf
SwitchCore.c:receiveMessage
InterfaceController.c:sendFromSwitch
UDPAddrIface:incomingFromIface

#0  incomingFromIface (m=0x7f05fa23fc38, iface=0x7f05fa1ba838) at util/events/libuv/UDPAddrIface.c:81
#1  0x00007f05f989a39f in incomingFromIface (m=<optimized out>, iface=<optimized out>) at util/events/libuv/UDPAddrIface.c:86
#2  0x00007f05f98a2da1 in Iface_send (msg=0x7f05fa23fc38, iface=<optimized out>) at ./interface/Iface.h:69
#3  sendFromSwitch (msg=0x7f05fa23fc38, switchIf=0x7f05fa1e4948) at net/InterfaceController.c:479
#4  0x00007f05f989e1da in Iface_send (msg=<optimized out>, iface=<optimized out>) at ./interface/Iface.h:69
#5  Iface_next (msg=<optimized out>, iface=<optimized out>) at ./interface/Iface.h:125
#6  receiveMessage (message=0x7f05fa23fc38, iface=0x0) at switch/SwitchCore.c:229
#7  0x00007f05f98b59cb in Iface_send (msg=0x7f05fa23fc38, iface=<optimized out>) at ./interface/Iface.h:69
#8  Iface_next (msg=0x7f05fa23fc38, iface=<optimized out>) at ./interface/Iface.h:125
#9  incomingFromSessionManagerIf (msg=0x7f05fa23fc38, sessionManagerIf=<optimized out>) at net/SwitchAdapter.c:47
#10 0x00007f05f98b12ed in Iface_send (msg=0x7f05fa23fc38, iface=0x90c56cbd6677c6e9) at ./interface/Iface.h:69
#11 Iface_next (msg=0x7f05fa23fc38, iface=0x90c56cbd6677c6e9) at ./interface/Iface.h:125
#12 readyToSend (msg=msg@entry=0x7f05fa23fc38, sm=sm@entry=0x7f05fa1d3398, sess=<optimized out>) at net/SessionManager.c:478
#13 0x00007f05f98b2ec5 in incomingFromInsideIf (msg=0x7f05fa23fc38, iface=0x7f05fa1d33b0) at net/SessionManager.c:518
#14 0x00007f05f98b5f5b in Iface_send (msg=0x7f05fa23fc38, iface=<optimized out>) at ./interface/Iface.h:69
#15 Iface_next (msg=0x7f05fa23fc38, iface=<optimized out>) at ./interface/Iface.h:125
#16 incomingFromTunAdapterIf (msg=0x7f05fa23fc38, tunAdapterIf=<optimized out>) at net/UpperDistributor.c:45
#17 0x00007f05f98b6eef in Iface_send (msg=<optimized out>, iface=<optimized out>) at ./interface/Iface.h:69
#18 Iface_next (msg=<optimized out>, iface=<optimized out>) at ./interface/Iface.h:125
#19 incomingFromTunIf (msg=0x7f05fa23fc38, tunIf=0x7f05fa1d3ba0) at net/TUNAdapter.c:52
#20 0x00007f05f989af6b in Iface_send (msg=0x7f05fa23fc38, iface=0x7f05fa1f9a48) at ./interface/Iface.h:69
#21 incoming (stream=<optimized out>, nread=<optimized out>, buf=<optimized out>) at util/events/libuv/Pipe.c:198
#22 0x00007f05f98d7c2f in uv__read (stream=stream@entry=0x7f05fa1f9b88) at ../src/unix/stream.c:1049
#23 0x00007f05f98d8358 in uv__stream_io (loop=<optimized out>, w=0x7f05fa1f9bf8, events=1) at ../src/unix/stream.c:1149
#24 0x00007f05f98dca64 in uv__io_poll (loop=loop@entry=0x7f05fa1b92b0, timeout=126) at ../src/unix/linux-core.c:284
#25 0x00007f05f98d1807 in uv_run (loop=0x7f05fa1b92b0, mode=mode@entry=UV_RUN_DEFAULT) at ../src/unix/core.c:284
#26 0x00007f05f986e3a5 in EventBase_beginLoop (eventBase=0x7f05fa1b9268) at util/events/libuv/EventBase.c:83
#27 0x00007f05f98b8510 in Core_main (argc=<optimized out>, argv=<optimized out>) at admin/angel/Core.c:333
#28 0x00007f05f9867503 in main (argc=3, argv=0x7fff821398e8) at client/cjdroute2.c:510
