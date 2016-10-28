## balance
+ /shared/opt/SCB/pos_eod/live/scripts/eod_task.pl -jobcode DSVAR03 -config irdfxdev9 
BASEDIR=/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0
2016_06_18 07:09:37 Loading job definition jobdefs/dsvar03.cf
2016_06_18 07:09:37 running job DSVAR03 (Sat 18 Jun 2016 @ 0709_37) on magapp15a (pid=21574)
2016_06_18 07:09:37 business processing date is 2015_04_30
2016_06_18 07:09:37 SNMP traps are ON
2016_06_18 07:09:37 post-processing checks are ON
2016_06_18 07:09:37 running job commands...
2016_06_18 07:09:37     trying to allocate session for user 
2016_06_18 07:09:37     Allocated session to user VARCON1 (ok)
2016_06_18 07:09:37     trying to allocate server 
2016_06_18 07:09:37     allocated session to run on server MAGAPP15A (ok)
2016_06_18 07:09:37     running cmd:
2016_06_18 07:09:37         cd /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client ; /shared/opt/jdk/pos/live/jre/bin/java
2016_06_18 07:09:37         -cp mxjboot.jar
2016_06_18 07:09:37         -Djava.security.policy=java.policy
2016_06_18 07:09:37         -Djava.rmi.server.codebase=http://uksvadmrx02a.uk.standardchartered.com:60001/murex.download.guiclient.\
download murex.rmi.loader.RmiLoader
2016_06_18 07:09:37         /MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript
2016_06_18 07:09:37         /MXJ_SITE_NAME:mxg_irdfx_dev9
2016_06_18 07:09:37         /MXJ_PLATFORM_NAME:MX
2016_06_18 07:09:37         /MXJ_PROCESS_NICK_NAME:POS_EOD_MXPS_MAGAPP15A_VARCON1
2016_06_18 07:09:37         /MXJ_CONFIG_FILE:/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_37.DSV\
AR03.psr/xml_request.xml
2016_06_18 07:09:37     end of cmd
2016_06_18 07:09:37     cmd std out:
2016_06_18 07:09:38         Opening bin/file.version...
2016_06_18 07:09:38         Opening jar/file.version...
2016_06_18 07:09:38         Checking files...
2016_06_18 07:09:38         bin/file.version saved.
2016_06_18 07:09:38         jar/file.version saved.
2016_06_18 07:09:39         Loading public/mxres/common/properties.mxres from http://uksvadmrx02a.uk.standardchartered.com:60001/mu\
rex.download.guiclient.download
2016_06_18 07:09:39         Loading locally /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_37.DSVA\
R03.psr/xml_request.xml.
2016_06_18 07:09:39         Loading public/mxres/sites/sites.mxres from http://uksvadmrx02a.uk.standardchartered.com:60001/murex.do\
wnload.guiclient.download
2016_06_18 07:09:39         murex.xml.server.home.balancing.exception.BalancingHomeException: Error while creating process for sess\
ion.Unable to find launcher for [MX,,POS_EOD_MXPS_MAGAPP15A_VARCON1]
2016_06_18 07:09:39             at murex.xml.server.home.balancing.BalancingLogic.logNotFoundLauncher(BalancingLogic.java:402)
2016_06_18 07:09:39             at murex.xml.server.home.balancing.BalancingLogic.createProcessForSession(BalancingLogic.java:109)
2016_06_18 07:09:39             at murex.xml.server.home.balancing.BalancingHome.createProcessForSession(BalancingHome.java:136)
2016_06_18 07:09:39             at murex.xml.server.home.XmlHome.createSessionLocally(XmlHome.java:592)
2016_06_18 07:09:39             at murex.xml.server.home.XmlHome.createSession(XmlHome.java:559)
2016_06_18 07:09:39             at sun.reflect.GeneratedMethodAccessor7.invoke(Unknown Source)
2016_06_18 07:09:39             at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
2016_06_18 07:09:39             at java.lang.reflect.Method.invoke(Method.java:324)
2016_06_18 07:09:39             at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:261)
2016_06_18 07:09:39             at sun.rmi.transport.Transport$1.run(Transport.java:148)
2016_06_18 07:09:39             at java.security.AccessController.doPrivileged(Native Method)
2016_06_18 07:09:39             at sun.rmi.transport.Transport.serviceCall(Transport.java:144)
2016_06_18 07:09:39             at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:460)
2016_06_18 07:09:39             at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:701)
2016_06_18 07:09:39             at java.lang.Thread.run(Thread.java:534)
2016_06_18 07:09:39     end of cmd std out
2016_06_18 07:09:39     cmd cd /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client ; /shared/opt/jdk/pos/live/jre/bin/java: ran with no\
rmal exit
2016_06_18 07:09:39     trying to de-allocate session for user VARCON1
2016_06_18 07:09:39     De-allocated session from user VARCON1 (ok)
2016_06_18 07:09:39     trying to de-allocate server MAGAPP15A
2016_06_18 07:09:39     de-allocated session from server MAGAPP15A (ok)
2016_06_18 07:09:39 end of job commands (last return code = 0)
2016_06_18 07:09:39 running post-processing checks for DSVAR03
2016_06_18 07:09:39     running level 1 check: AC1 fileExists ( '/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30\
/eod_job.0709_37.DSVAR03.psr/answer.xml'  )
2016_06_18 07:09:39     end of check (ok)
2016_06_18 07:09:39     running level 2 check: AC2 searchFile ( '/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30\
/eod_job.0709_37.DSVAR03.psr/answer.xml' 'present' '-1' '-1' 'Ended_Successfully'  )
2016_06_18 07:09:39         ERR: Batch failed. See /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_\
37.DSVAR03.psr/answer.xml.
2016_06_18 07:09:39         no SNMP rasied
2016_06_18 07:09:39     end of check (failed)
2016_06_18 07:09:39     no SNMP rasied
2016_06_18 07:09:39     sending mail: job DSVAR03 (started on Sat 18 Jun 2016 @ 0709_37) failed level 2 check [ AC2 searchFile ( '/\
shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709_37.DSVAR03.psr/answer.xml' 'present' '-1' '-1' 'Ende\
d_Successfully'  ) ] see logfile [ eod_job.0709_37.DSVAR03.log ] err [ Batch failed. See /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/c\
lient/runtime/2015_04_30/eod_job.0709_37.DSVAR03.psr/answer.xml. ] feedback [ ]
2016_06_18 07:09:39     Production domain is gdc.standardchartered.com, this domain is uk.standardchartered.com
2016_06_18 07:09:39 end of post-processing checks for DSVAR03 (failed)
2016_06_18 07:09:39 --- 1 -----------------------
"job DSVAR03 (started on Sat 18 Jun 2016 @ 0709_37) failed level 2 check [ AC2 searchFile ( '/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1\
.0/client/runtime/2015_04_30/eod_job.0709_37.DSVAR03.psr/answer.xml' 'present' '-1' '-1' 'Ended_Successfully'  ) ] see logfile [ eo\
d_job.0709_37.DSVAR03.log ] err [ Batch failed. See /shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/client/runtime/2015_04_30/eod_job.0709\
_37.DSVAR03.psr/answer.xml. ] feedback [ ]"
Error Counts -----------------------------------
Critical: 1
All:      1

2016_06_18 07:09:39 end of job DSVAR03 (failed)
2016_06_18 07:09:39 job DSVAR03 took  2 wallclock secs ( 0.02 usr  0.05 sys +  1.00 cusr  0.31 csys =  1.38 CPU)


## ## putty ##
Mai_title: Emailing: putty download method.zip  / FW: Tools Share 

## Aqua Data Studio ##



## Set up VPN software client## 
Mail_Title: Follow Up - 8308129-001-201 - VPN Setup

## Set up Printer Driver## 
Mail_Title: RMS# 8327306-007-101 add printer driver
Doc: Job\Document\Set_up_printer

## Outlook ## Add WenJun & Tony ( Zhu Bing ) into mail grp 
Firstly make sure you have store the group as your contact, then open the group contact and then double click the group name and pop out a pannel and then click the 'modify number'.

 
## Timesheet ## Submit timesheet in WorkWise.
Doc: Job\Document\TimeSheet
	

## Self-Regist in JIRA ##
https://jira.uk.standardchartered.com:8443/secure/Dashboard.jspa (Note visit it with Chrome not IE,and the url is incorrect in Job\Document\SetupDevelopmentEnvironment\New Joiners.docx)  


## VPN don't work ##
check Preferences->Block connections to untrusted server(check out this item)


## Control-M ????????????????????????????
ITIL(Information Technology Infrastructure Library),信息技术基础架构库，由英国政府部门CCTA(Central Computing and Telecomunications Agency)在20世纪80年代末制定，现由英国商务部（0
BSM(Business Service Management),业务服务管理，
Control-M production is developed by BMC software Corporation ,


## Excel - vlookup ??????????????????????
