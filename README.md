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



## murex client 
1)bin  文件夹
  avsjogl.dll
  xxx.dll
  bridge2mx.tlb
  excel2mx.tlb
  file.version
  xxx.dll
  regtlb.exe
2)jar 文件夹
 　sun.jar
  com.ibm.mq.jar
  com.ibm.mqbind.jar
  com.ibm.mqjms.jar
  excel.jar
  ie.jar
  file.version
  jta.jar
  mail.jar
  mxj.jar
  servlet.jar
  word.jar
  xml4.jar
  xxxx.jar

3)mxjboot.jar
4)client_mxg_qa1.bat
  
@ECHO OFF

REM Mx G2000 Client Launcher
REM Mofify this script to match your java and server environnement
REM For 2.2.8 and 2.2.9
REM V2.3

setlocal

SET JAVAHOME=%ProgramFiles(x86)%\Java\jre1.5.0_15

SET MXJ_FILESERVER_HOST=magapp11a.uk.standardchartered.com
SET MXJ_FILESERVER_PORT=14331
SET MXJ_SITE_NAME=default
SET MXJ_DESTINATION_SITE_NAME=mxg_qa1
SET MXJ_PLATFORM_NAME=MX
SET MXJ_PROCESS_NICK_NAME=MXG_QA1


SET PATH=%JAVAHOME%\jre\bin;%JAVAHOME%\jre\bin\classic;%JAVAHOME%\bin;%JAVAHOME%\bin\classic;%PATH%

SET PATH=%PATH%;bin
SET MXJ_JAR_FILELIST=murex.download.guiclient.download
SET MXJ_POLICY=java.policy
SET MXJ_BOOT=mxjboot.jar
SET MXJ_CONFIG_FILE=client.xml

IF EXIST jar\%MXJ_BOOT% copy jar\%MXJ_BOOT% . >NUL

title %~n0 FS:%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST%  Xml:%MXJ_SITE_NAME% /PLATF:%MXJ_PLATFORM_NAME% /NNAME:%MXJ_PROCESS_NICK_NAME%

java -Xmx512m -cp %MXJ_BOOT% -Djava.security.policy=%MXJ_POLICY% -Djava.rmi.server.codebase=http://%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST% murex.rmi.loader.RmiLoader /MXJ_SITE_NAME:%MXJ_SITE_NAME% /MXJ_DESTINATION_SITE_NAME:%MXJ_DESTINATION_SITE_NAME% /MXJ_CLASS_NAME:murex.gui.xml.XmlGuiClientBoot /MXJ_PLATFORM_NAME:%MXJ_PLATFORM_NAME% /MXJ_PROCESS_NICK_NAME:%MXJ_PROCESS_NICK_NAME% /MXJ_CONFIG_FILE:%MXJ_CONFIG_FILE% %1 %2 %3 %4 %5 %6

title Command Prompt
endlocal
pause

## mxjboot.jar
package murex.rmi.loader.parser.sax;

class Attribute
{
  private String name;
  private String value;

  public Attribute(String n, String v)
  {
    this.name = n;
    this.value = v;
  }

  public String getName()
  {
    return this.name;
  }

  public String getValue()
  {
    return this.value;
  }
}

package murex.rmi.loader.parser.sax;

import java.util.Vector;

public class AttributeList
{
  private Vector attributes;

  public AttributeList()
  {
    this.attributes = new Vector();
  }

  public void addAttribute(String name, String value)
  {
    Attribute attr = new Attribute(name, value);
    this.attributes.addElement(attr);
  }

  public int getLength()
  {
    return this.attributes.size();
  }

  public String getName(int index)
  {
    return ((Attribute)this.attributes.elementAt(index)).getName();
  }

  public String getValue(String n)
  {
    int i = getIndex(n);
    if (i != -1) {
      return getValue(i);
    }
    return null;
  }

  public String getValue(int index)
  {
    return ((Attribute)this.attributes.elementAt(index)).getValue();
  }

  public String getType(int index)
  {
    return null;
  }

  public String getType(String n)
  {
    return null;
  }

  public void clear()
  {
    this.attributes.removeAllElements();
  }

  private int getIndex(String n)
  {
    int index = -1;
    for (int i = 0; i < this.attributes.size(); i++)
      if (((Attribute)this.attributes.elementAt(i)).getName().equals(n))
        index = i;
    return index;
  }
}

package murex.rmi.loader.parser.sax;

public abstract interface HandlerBase
{
  public abstract void characters(char[] paramArrayOfChar, int paramInt1, int paramInt2);

  public abstract void startDocument();

  public abstract void endDocument();

  public abstract void startElement(String paramString, AttributeList paramAttributeList);

  public abstract void endElement(String paramString);
}

package murex.rmi.loader.parser.sax;

import java.io.InputStream;
import java.io.Reader;

public class InputSource
{
  private InputStream byteStream;
  private Reader characterStream;

  public InputSource(InputStream i)
  {
    this.byteStream = i;
  }

  public InputSource(Reader r)
  {
    this.characterStream = r;
  }
}

package murex.rmi.loader.parser.sax;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;

public class Parser
{
  private HandlerBase handler = null;
  private char[] input;
  private int cursor = 0;
  private int length = 0;
  private String elemName;
  private boolean endElem = false;
  private AttributeList attributes;

  public void setDocumentHandler(HandlerBase h)
  {
    this.handler = h;
  }

  public void parse(String str)
  {
    parse(str.toCharArray());
  }

  public void parse(Reader r) throws IOException
  {
    BufferedReader bf = new BufferedReader(r);

    String str = new String();
    String str1;
    while ((str1 = bf.readLine()) != null)
    {
      str1 = str1.trim();
      str = str + str1;
    }
    char[] ch = str.toCharArray();
    parse(ch);
  }

  public void parse(char[] data) {
    this.input = data;
    int prevCursor = 0;
    this.length = this.input.length;
    boolean foundChar = false;
    int charCur = 0;

    this.handler.startDocument();

    for (this.cursor = 0; this.cursor < this.length; this.cursor += 1) {
      switch (this.input[this.cursor]) {
      case '&':
        gotAmpersand();
        break;
      case '<':
        if (this.cursor > prevCursor)
          this.handler.characters(this.input, prevCursor, this.cursor - prevCursor);
        if ((this.input[(this.cursor + 1)] == '?') || (this.input[(this.cursor + 1)] == '!')) {
          ignore();
          prevCursor = this.cursor + 1;
        }
        else {
          gotLT();
        }break;
      case '>':
        foundChar = false;
        prevCursor = this.cursor + 1;
        gotGT();
        if (this.input[(this.cursor - 1)] == '/')
          this.handler.endElement(this.elemName); break;
      case '\n':
        prevCursor = this.cursor + 1;
      }

    }

    this.handler.endDocument();
  }

  private void gotLT()
  {
    this.attributes = null;
    boolean b = false;
    boolean gotName = false;
    int prevCursor = this.cursor + 1;

    this.endElem = false;
    for (this.cursor += 1; this.cursor < this.length; this.cursor += 1)
    {
      switch (this.input[this.cursor])
      {
      case '&':
        gotAmpersand();
        break;
      case '/':
        if (this.input[(this.cursor + 1)] != '>')
        {
          prevCursor++;
          this.endElem = true; } break;
      case '>':
        b = true;
        if (!gotName)
        {
          if (this.input[(this.cursor - 1)] != '/')
            this.elemName = new String(this.input, prevCursor, this.cursor - prevCursor);
          else
            this.elemName = new String(this.input, prevCursor, this.cursor - prevCursor - 1);
          gotName = true;
        }
        this.cursor -= 1;
        break;
      case ' ':
        if (!gotName)
        {
          this.elemName = new String(this.input, prevCursor, this.cursor - prevCursor);
          gotName = true;
        }
        gotAttribute();
      }

      if (b)
        break;
    }
  }

  private void gotAttribute() {
    if (this.attributes == null) {
      this.attributes = new AttributeList();
    }
    int prevCursor = this.cursor + 1;
    int i = 0;
    boolean b = false;

    for (this.cursor += 1; this.cursor < this.length; this.cursor += 1)
    {
      switch (this.input[this.cursor])
      {
      case '&':
        gotAmpersand();
        break;
      case '=':
        i = this.cursor;
        break;
      case ' ':
        b = true;
        this.cursor -= 1;
        break;
      case '>':
        b = true;
        this.cursor -= 1;
        if (this.input[this.cursor] == '/')
          this.cursor -= 1;
        break;
      }
      if (b)
        break;
    }
    String n = new String(this.input, prevCursor, i - prevCursor);
    String v;
    String v;
    if (this.cursor - i > 2)
      v = new String(this.input, i + 2, this.cursor - i - 2);
    else
      v = new String("");
    this.attributes.addAttribute(n, v);
  }

  private void gotGT()
  {
    if (!this.endElem)
      this.handler.startElement(this.elemName, this.attributes);
    else
      this.handler.endElement(this.elemName);
  }

  private void ignore()
  {
    for (this.cursor += 1; (this.cursor < this.length) && 
      (this.input[this.cursor] != '>'); this.cursor += 1);
  }

  private void gotAmpersand()
  {
    int i = 0;
    char[] ch = new char[5];
    while (this.input[(i + this.cursor + 1)] != ';')
    {
      ch[i] = this.input[(i + this.cursor + 1)];
      i++;
    }
    String str = new String(ch, 0, i);

    if (str.equals("amp"))
      this.input[this.cursor] = '&';
    if (str.equals("quot"))
      this.input[this.cursor] = '"';
    if (str.equals("lt"))
      this.input[this.cursor] = '<';
    if (str.equals("gt"))
      this.input[this.cursor] = '>';
    if (str.equals("apos"))
      this.input[this.cursor] = '\'';
    System.arraycopy(this.input, i + this.cursor + 2, this.input, this.cursor + 1, this.length - i - this.cursor - 2);
    this.length = (this.length - i - 1);
  }
}

package murex.rmi.loader.parser;

public class CheckFile
{
  private String strFile;
  private String strCheck;
  private String strVersion;
  private String strCheckSum;

  public CheckFile(String strFile, String strCheck, String strVersion, String strCheckSum)
  {
    this.strFile = strFile;
    this.strCheck = strCheck;
    this.strVersion = strVersion;
    this.strCheckSum = strCheckSum;
  }

  public String getCheck() {
    return this.strCheck;
  }

  public String getFile() {
    return this.strFile;
  }

  public String getVersion() {
    return this.strVersion;
  }

  public String getCheckSum() {
    return this.strCheckSum;
  }

  public String toString() {
    return "[File : " + this.strFile + (this.strCheck == null ? "" : new StringBuffer().append(" , Check : ").append(this.strCheck).toString()) + (this.strVersion == null ? "" : new StringBuffer().append(" , Version : ").append(this.strVersion).toString()) + (this.strCheckSum == null ? "" : new StringBuffer().append(" , CheckSum : ").append(this.strCheckSum).toString()) + "]";
  }
}

package murex.rmi.loader.parser;

public class FileVersion
{
  private String strFile;
  private String strVersion;
  private String strCheckSum;

  public FileVersion(String strFile, String strVersion, String strCheckSum)
  {
    this.strFile = strFile;
    this.strVersion = strVersion;
    this.strCheckSum = strCheckSum;
  }

  public String getFile() {
    return this.strFile;
  }

  public String getVersion() {
    return this.strVersion;
  }

  public String getCheckSum() {
    return this.strCheckSum;
  }

  public String toString() {
    return "[File : " + this.strFile + (this.strVersion == null ? " , CheckSum : " + this.strCheckSum : new StringBuffer().append(" , Version : ").append(this.strVersion).toString()) + "]";
  }
}

package murex.rmi.loader.parser;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.util.ArrayList;
import murex.rmi.loader.parser.sax.AttributeList;
import murex.rmi.loader.parser.sax.HandlerBase;

public class Parser
  implements HandlerBase
{
  public static final String tagPhysicalDownLoad = "PhysicalDownLoad";
  public static final String tagMemoryDownLoad = "MemoryDownLoad";
  public static final String tagClassFiles = "ClassFiles";
  public static final String tagCheck = "Check";
  public static final String tagFile = "File";
  public static final String tagVersion = "Version";
  public static final String tagCheckSum = "CheckSum";
  private murex.rmi.loader.parser.sax.Parser parser;
  private ArrayList physical;
  private ArrayList memory;
  private ArrayList currentDld;
  private String currentTag;
  private String currentFile;
  private String currentCheck;
  private String currentVersion;
  private String currentCheckSum;

  public Parser(String strAllFile)
    throws Exception
  {
    this.parser = new murex.rmi.loader.parser.sax.Parser();
    this.parser.setDocumentHandler(this);
    this.parser.parse(strAllFile);
  }

  public void characters(char[] ch, int start, int length) {
    this.currentFile = new String(ch, start, length);
  }

  public void startDocument() {
  }

  public void endDocument() {
  }

  public void startElement(String name, AttributeList attributes) {
    if (name.equals("PhysicalDownLoad")) {
      this.physical = new ArrayList();
      this.currentTag = "PhysicalDownLoad";
      this.currentDld = this.physical;
    } else if (name.equals("ClassFiles")) {
      this.memory = new ArrayList();
      this.currentDld = this.memory;
      this.currentTag = "MemoryDownLoad";
    } else if ((name.equals("File")) && 
      (attributes != null)) {
      this.currentCheck = attributes.getValue("Check");
      this.currentVersion = attributes.getValue("Version");
      this.currentCheckSum = attributes.getValue("CheckSum");
    }
  }

  public void endElement(String name)
  {
    if (name.equals("File"))
      this.currentDld.add(new CheckFile(this.currentFile, this.currentCheck, this.currentVersion, this.currentCheckSum));
  }

  public CheckFile[] getPhysicalDownload()
  {
    if (this.physical != null) {
      return (CheckFile[])this.physical.toArray(new CheckFile[0]);
    }
    return new CheckFile[0];
  }

  public CheckFile[] getMemoryDownload() {
    if (this.memory != null) {
      return (CheckFile[])this.memory.toArray(new CheckFile[0]);
    }
    return new CheckFile[0];
  }

  public static void main(String[] argv) {
    try {
      StringBuffer sb = new StringBuffer();
      BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(argv[0])));
      String line = br.readLine();
      while (line != null) {
        sb.append(line);
        line = br.readLine();
      }
      Parser parser = new Parser(sb.toString());

      System.out.println("PhysicalDownload:");
      CheckFile[] physical = parser.getPhysicalDownload();
      if (physical != null) {
        for (int i = 0; i < physical.length; i++) {
          System.out.println(physical[i]);
        }
      }
      System.out.println("MemoryDownload:");
      CheckFile[] memory = parser.getMemoryDownload();
      if (memory != null)
        for (int i = 0; i < memory.length; i++)
          System.out.println(memory[i]);
    }
    catch (Exception e) {
      e.printStackTrace();
    }
  }
}

package murex.rmi.loader.parser;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;
import murex.rmi.loader.parser.sax.AttributeList;
import murex.rmi.loader.parser.sax.HandlerBase;
import murex.rmi.loader.parser.sax.Parser;

public class VersionParser
  implements HandlerBase
{
  public String tagFiles = "Files";
  public String tagFile = "File";
  public String tagVersion = "Version";
  public String tagCheckSum = "CheckSum";
  private Parser parser;
  private Map map;
  private String currentFile;
  private String currentVersion;
  private String currentCheckSum;

  public VersionParser(String strFile)
    throws Exception
  {
    StringBuffer sb = new StringBuffer();
    BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(strFile)));
    String line = br.readLine();
    while (line != null) {
      sb.append(line);
      line = br.readLine();
    }

    this.map = new TreeMap();
    this.parser = new Parser();
    this.parser.setDocumentHandler(this);
    this.parser.parse(sb.toString());
  }

  public void characters(char[] ch, int start, int l) {
    this.currentFile = new String(ch, start, l);
  }

  public void startDocument() {
  }

  public void endDocument() {
  }

  public void startElement(String name, AttributeList attributes) {
    if ((name.equals(this.tagFile)) && 
      (attributes != null)) {
      this.currentVersion = attributes.getValue(this.tagVersion);
      this.currentCheckSum = attributes.getValue(this.tagCheckSum);
    }
  }

  public void endElement(String name)
  {
    if (name.equals(this.tagFile)) {
      FileVersion fileVersion = new FileVersion(this.currentFile, this.currentVersion, this.currentCheckSum);
      this.map.put(fileVersion.getFile(), fileVersion);
      this.currentVersion = null;
      this.currentCheckSum = null;
    }
  }

  public Map getFileVersion() {
    return this.map;
  }

  public static void main(String[] argv) {
    try {
      VersionParser versionParser = new VersionParser(argv[0]);
      System.out.println("File : ");
      Map map = versionParser.getFileVersion();
      Iterator iterator = map.keySet().iterator();
      while (iterator.hasNext()) {
        String strFile = (String)iterator.next();
        System.out.println(map.get(strFile));
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}

package murex.rmi.loader.start;

import java.io.PrintStream;

public class FileServerAutomaticDownload
  implements Runnable
{
  public static void main(String[] args)
  {
    System.out.println("MX files were successfully downloaded...");
  }
  public void run() {
    main(null);
  }
}

package murex.rmi.loader.webstart;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLConnection;

public class BuildClientDir
{
  public static final String tagFile = "File";
  public static final String tagException = "EXCEPTION";
  public static final String tagMurexAnswer = "Murex-Answer";
  public static final String FILE_SERVER = "/FILE_SERVER=";
  public static final String FILE_SERVER_PORT = "/FILE_SERVER_PORT=";
  public static final String MXDOC_SERVER = "/MXDOC_SERVER=";
  public static final String MXDOC_SERVER_PORT = "/MXDOC_SERVER_PORT=";
  public static final String MXDOC_HTTP_PROXY_HOST = "/MXDOC_HTTP_PROXY_HOST=";
  public static final String MXDOC_HTTP_PROXY_PORT = "/MXDOC_HTTP_PROXY_PORT=";
  public static final String MXDOC_PROXY_AUTHENTIFICATION_NAME = "/MXDOC_PROXY_AUTHENTIFICATION_NAME=";
  public static final String MXDOC_PROXY_AUTHENTIFICATION_PASSWD = "/MXDOC_PROXY_AUTHENTIFICATION_PASSWD=";
  public static final String CLIENT_DIR = "/CLIENT_DIR=";
  public static final String JAVA_SECURITY_POLICY = "-Djava.security.policy=";
  public static final String RMI_SERVER_CODE_BASE = "-Djava.rmi.server.codebase=";
  public static final String MXJ_BOOT_URL_FROM_FS = "webclient/jar/mxjboot.jar";
  public static final String MXJ_MXDOC_BOOT_URL_FROM_FS = "webclient/mxdoc/mxjboot.jar";
  public static final String MXJ_JAVA_HOME = "JAVA_HOME=";
  public static final String MXJ_JAVA_MAX_HEAP = "JAVA_MAX_HEAPSIZE=";
  public static final String MXJ_JAVA_MIN_HEAP = "JAVA_MIN_HEAPSIZE=";

  public static void main(String[] argv)
  {
    if (argv.length >= 16)
      try {
        BuildClientDir buildClientDir = new BuildClientDir();
        buildClientDir.getBoot(argv);
        buildClientDir.getMxDocBoot(argv);
        File clientCmdFile = buildClientDir.createClientCmdFile(argv);
        buildClientDir.createMxDocCmdFile(argv);
        buildClientDir.launchWebClient(clientCmdFile);
      } catch (Exception e) {
        e.printStackTrace();
      }
    else
      System.out.println("Please Enter the minimum number of arguments in the JNLP file...");
  }

  private void getBoot(String[] strArgs)
    throws Exception
  {
    String fileServerHost = getArgument(strArgs, "/FILE_SERVER=");
    String fileServerPort = getArgument(strArgs, "/FILE_SERVER_PORT=");
    String jnlpClientPath = getArgument(strArgs, "/CLIENT_DIR=");
    String clientPath = computeClientDirPath(jnlpClientPath);
    System.out.println("/FILE_SERVER=" + fileServerHost);
    System.out.println("/FILE_SERVER_PORT=" + fileServerPort);
    System.out.println("/CLIENT_DIR=" + clientPath);

    String strMxjBootUrl = "http://" + fileServerHost + ":" + fileServerPort + "/" + "webclient/jar/mxjboot.jar";
    URL servUrl = new URL(strMxjBootUrl);
    try
    {
      jarFile = getFileFromFileServer(servUrl);
    }
    catch (Exception e)
    {
      byte[] jarFile;
      e.printStackTrace(System.err);
      throw e;
    }
    byte[] jarFile;
    File clientPathFile = new File(clientPath);
    if (!clientPathFile.exists()) {
      if (clientPathFile.mkdirs())
        System.out.println("The folder CLIENT_DIR has been created: " + clientPathFile);
      else {
        throw new Exception("Failed to create CLIENT_DIR: " + clientPathFile);
      }
    }

    String destJarFile = clientPath + "\\mxjboot.jar";
    createMxjBoot(jarFile, destJarFile);
  }

  private String computeClientDirPath(String jnlpPath) throws Exception
  {
    String returnedString = null;
    String rootClientPath = null;
    String userName = System.getProperty("user.name");
    if (jnlpPath.indexOf("\\USER.NAME") == -1)
    {
      returnedString = jnlpPath;
    }
    else {
      rootClientPath = jnlpPath.substring(0, jnlpPath.indexOf("\\USER.NAME"));
      returnedString = rootClientPath + "\\" + userName;
    }
    return returnedString;
  }

  private void getMxDocBoot(String[] strArgs) throws Exception {
    String fileServerHost = getArgument(strArgs, "/FILE_SERVER=");
    String fileServerPort = getArgument(strArgs, "/FILE_SERVER_PORT=");
    String jnlpClientPath = getArgument(strArgs, "/CLIENT_DIR=");
    String clientPath = computeClientDirPath(jnlpClientPath);
    String mxDocClientPath = clientPath + "\\mxdoc";

    String strMxjMxDocBootUrl = "http://" + fileServerHost + ":" + fileServerPort + "/" + "webclient/mxdoc/mxjboot.jar";
    URL servUrl = new URL(strMxjMxDocBootUrl);
    try
    {
      jarFile = getFileFromFileServer(servUrl);
    }
    catch (Exception e)
    {
      byte[] jarFile;
      e.printStackTrace(System.err);
      throw e;
    }
    byte[] jarFile;
    File mxDocClientPathFile = new File(mxDocClientPath);
    if (!mxDocClientPathFile.exists()) {
      if (mxDocClientPathFile.mkdirs())
        System.out.println("The folder mxdoc has been created: " + mxDocClientPathFile);
      else {
        throw new Exception("Failed to create mxdoc: " + mxDocClientPathFile);
      }
    }

    String destJarFile = mxDocClientPath + "\\mxjboot.jar";
    createMxjBoot(jarFile, destJarFile);
  }

  private static byte[] getFileFromFileServer(URL url) throws Exception {
    String strRequest = url.getFile() + "?" + "File" + "=" + "webclient/jar/mxjboot.jar";
    URL urlToFileServer = new URL(url.getProtocol(), url.getHost(), url.getPort(), strRequest);
    URLConnection urlConnection = urlToFileServer.openConnection();
    urlConnection.setUseCaches(false);
    urlConnection.setRequestProperty("MUREX-EXTENSION", "murex/murex.rmi.loader.FileServerGenericLoader");
    InputStream inputStream = urlConnection.getInputStream();
    if (urlConnection.getContentLength() == -1) {
      throw new Exception("unable to get file webclient/jar/mxjboot.jar from " + url + ".");
    }

    String strStatus = urlConnection.getHeaderField("Murex-Answer");
    if ("EXCEPTION".equals(strStatus)) {
      byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
      throw new Exception(new String(inputFile));
    }
    byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
    inputStream.close();
    return inputFile;
  }

  private static byte[] getBodyStream(InputStream inputStream, int count) {
    byte[] tab = new byte[count];
    try {
      int iReceived = 0;
      int iTempReceived = 0;
      while ((iTempReceived = inputStream.read(tab, iReceived, count - iReceived)) > 0)
        iReceived += iTempReceived;
    }
    catch (IOException e) {
    }
    return tab;
  }

  private void createMxjBoot(byte[] byteFile, String strDestFile) {
    if (byteFile.length != 0)
      try {
        FileOutputStream fos = new FileOutputStream(strDestFile);
        fos.write(byteFile);
        fos.close();
        System.out.println(strDestFile + " copied.");
      } catch (Exception e) {
        System.out.println(e.getMessage());
      }
  }

  private static String getArgument(String[] strArgs, String neededArg)
  {
    for (int i = 0; i < strArgs.length; i++) {
      if (strArgs[i].startsWith(neededArg)) {
        return strArgs[i].substring(neededArg.length());
      }
    }
    return null;
  }

  public File createClientCmdFile(String[] args) throws Exception {
    String jnlpClientPath = getArgument(args, "/CLIENT_DIR=");
    String destDir = computeClientDirPath(jnlpClientPath);
    File webclientCmd = new File(destDir, "webclient.cmd");
    String javaHome = System.getProperty("java.home");
    String javaSecPolicy = "";
    String commandArgs = "";
    String rmiServerCodeBase = "";
    String strXmx = null;
    String strXms = null;

    for (int i = 0; i < args.length; i++) {
      String arg = args[i];
      if (arg.startsWith("-Djava.security.policy="))
        javaSecPolicy = arg;
      else if (arg.startsWith("-Djava.rmi.server.codebase="))
        rmiServerCodeBase = arg;
      else if (arg.startsWith("JAVA_HOME="))
        javaHome = arg.substring("JAVA_HOME=".length());
      else if (arg.startsWith("JAVA_MAX_HEAPSIZE="))
        strXmx = arg.substring("JAVA_MAX_HEAPSIZE=".length());
      else if (arg.startsWith("JAVA_MIN_HEAPSIZE="))
        strXms = arg.substring("JAVA_MIN_HEAPSIZE=".length());
      else if ((arg.indexOf("/FILE_SERVER=") == -1) && (arg.indexOf("/FILE_SERVER_PORT=") == -1) && (arg.indexOf("/CLIENT_DIR=") == -1) && (arg.indexOf("/MXDOC_HTTP_PROXY_HOST=") == -1) && (arg.indexOf("/MXDOC_HTTP_PROXY_PORT=") == -1) && (arg.indexOf("-Djava") == -1))
      {
        commandArgs = commandArgs + " " + arg;
      }
    }

    String javaCmd = "java ";
    if (strXmx != null) {
      javaCmd = javaCmd + "-Xmx" + strXmx + " ";
    }
    if (strXms != null) {
      javaCmd = javaCmd + "-Xms" + strXms + " ";
    }
    javaCmd = javaCmd + "-cp mxjboot.jar " + javaSecPolicy + " " + rmiServerCodeBase + " murex.rmi.loader.RmiLoader" + commandArgs;

    String[] cmdFile = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION", "REM Please do not modify this file.", "", "set JAVAHOME=" + javaHome, "set PATH=%JAVAHOME%\\jre\\bin;%JAVAHOME%\\jre\\bin\\client;%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", javaCmd, "", "endlocal" };

    writeToFile(cmdFile, webclientCmd);

    return webclientCmd;
  }

  public void createMxDocCmdFile(String[] args) throws Exception {
    String mxDocServerHost = getArgument(args, "/MXDOC_SERVER=");
    String mxDocServerPort = getArgument(args, "/MXDOC_SERVER_PORT=");
    String mxDocHttpProxyHost = getArgument(args, "/MXDOC_HTTP_PROXY_HOST=");
    String mxDocHttpProxyPort = getArgument(args, "/MXDOC_HTTP_PROXY_PORT=");
    String mxDocProxyAuthentificationName = getArgument(args, "/MXDOC_PROXY_AUTHENTIFICATION_NAME=");
    String mxDocProxyAuthentificationPass = getArgument(args, "/MXDOC_PROXY_AUTHENTIFICATION_PASSWD=");
    String jnlpClientPath = getArgument(args, "/CLIENT_DIR=");
    String destDir = computeClientDirPath(jnlpClientPath).concat("\\mxdoc");
    File jmdbrowserCmd = new File(destDir, "jmdbrowser.cmd");
    String javaHome = System.getProperty("java.home");
    String javaSecPolicy = "";
    String rmiServerCodeBase = "";

    for (int i = 0; i < args.length; i++) {
      String arg = args[i];
      if (arg.startsWith("-Djava.security.policy=")) {
        javaSecPolicy = arg;
      }
      if (arg.startsWith("-Djava.rmi.server.codebase=")) {
        rmiServerCodeBase = arg;
      }
    }

    String[] cmdFileNoProxy = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION.", "", "REM Do not remove the following line", "cd mxdoc", "", "set JAVAHOME=" + javaHome, "REM  (US character code)", "chcp 437", "set PATH=%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", "REM No HTTP Proxy used", "REM java -Xms256M -Xmx512M -Xss16M -cp ...", "java -cp mxjboot.jar " + javaSecPolicy + " -Djava.rmi.server.codebase=http://" + mxDocServerHost + ":" + mxDocServerPort + "/murex.download.jmdbrowser.download murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.JMDBrowser.JMDBrowser -fs:" + mxDocServerHost + ":" + mxDocServerPort + " %1 %2 %3 %4 %5 %6", "", "endlocal" };

    String[] cmdFileProxy = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION.", "", "REM Do not remove the following line", "cd mxdoc", "", "set JAVAHOME=" + javaHome, "REM  (US character code)", "chcp 437", "set PATH=%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", "REM HTTP Proxy used without authentification", "java -Dhttp.proxyHost=" + mxDocHttpProxyHost + " -Dhttp.proxyPort=" + mxDocHttpProxyPort + " -cp mxjboot.jar -Djava.security.policy=" + javaSecPolicy + " -Djava.rmi.server.codebase=http://" + mxDocServerHost + ":" + mxDocServerPort + "/murex.download.jmdbrowser.download murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.JMDBrowser.JMDBrowser -fs:" + mxDocServerHost + ":" + mxDocServerPort + " %1 %2 %3 %4 %5 %6", "", "endlocal" };

    String[] cmdFileProxyAuthentification = { "@ECHO OFF", "setlocal", "", "REM This file was created by the MUREX JWS WEB CLIENT APPLICATION.", "", "REM Do not remove the following line", "cd mxdoc", "", "set JAVAHOME=" + javaHome, "REM  (US character code)", "chcp 437", "set PATH=%JAVAHOME%\\bin;%JAVAHOME%\\bin\\client;%PATH%", "set PATH=%PATH%;" + destDir + File.separator + "bin\\", "", "REM HTTP Proxy used with authentification", "java -Dhttp.proxyHost=" + mxDocHttpProxyHost + " -Dhttp.proxyPort=" + mxDocHttpProxyPort + " -Dmurex.proxy.username=" + mxDocProxyAuthentificationName + " -Dmurex.proxy.password=" + mxDocProxyAuthentificationPass + " -cp mxjboot.jar -Djava.security.policy=" + javaSecPolicy + " -Djava.rmi.server.codebase=http://" + mxDocServerHost + ":" + mxDocServerPort + "/murex.download.jmdbrowser.download murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.JMDBrowser.JMDBrowser -fs:" + mxDocServerHost + ":" + mxDocServerPort + " %1 %2 %3 %4 %5 %6", "", "endlocal" };

    if (mxDocHttpProxyHost.equals("%MXDOC_HTTP_PROXY_HOST_NAME%")) {
      writeToFile(cmdFileNoProxy, jmdbrowserCmd);
    }
    else if (mxDocProxyAuthentificationName.equals("%MXDOC_PROXY_AUTHENTIFICATION_NAME%")) {
      writeToFile(cmdFileProxy, jmdbrowserCmd);
    }
    else
      writeToFile(cmdFileProxyAuthentification, jmdbrowserCmd);
  }

  private static void writeToFile(String[] lines, File file)
  {
    System.out.println("Build " + file.getAbsolutePath());

    if (file.exists()) {
      file.delete();
    }
    try
    {
      pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
    }
    catch (IOException e)
    {
      PrintWriter pw;
      throw new RuntimeException("Failed to create writer to file: " + file);
    }
    try
    {
      for (int i = 0; i < lines.length; i++)
        pw.println(lines[i]);
      pw.flush();
    }
    finally
    {
      PrintWriter pw;
      pw.close();
    }
  }

  void launchWebClient(File webClientCmdFile)
  {
    File workingDir = webClientCmdFile.getParentFile();
    try
    {
      System.out.println("Exec: " + webClientCmdFile + ", in working dir: " + workingDir);
      xProcess = Runtime.getRuntime().exec(webClientCmdFile.getAbsolutePath(), null, workingDir); } catch (IOException e) { Process xProcess;
      System.err.println("Failure executing process: " + webClientCmdFile.getAbsolutePath() + ", in working dir: " + workingDir);
      e.printStackTrace(System.err);
      return; }
    Process xProcess;
    File logFile = new File(workingDir, "jws.log");
    System.out.println("Logging process output to file: " + logFile);
    PrintWriter logFileWriter;
    try { PrintWriter logFileWriter = new PrintWriter(new FileOutputStream(logFile));

      ReaderWriterPipe stdoutReaderWriter = new ReaderWriterPipe(new BufferedReader(new InputStreamReader(xProcess.getInputStream())), logFileWriter);
      ReaderWriterPipe stderrReaderWriter = new ReaderWriterPipe(new BufferedReader(new InputStreamReader(xProcess.getErrorStream())), logFileWriter);

      new Thread(stdoutReaderWriter, "stdoutReader").start();
      new Thread(stderrReaderWriter, "stderrReader").start();
    } catch (FileNotFoundException e) {
      System.err.println("Failed to write to log file: " + logFile);
      e.printStackTrace(System.err);
      logFileWriter = null;
    }
    try
    {
      xProcess.waitFor();
    } catch (Exception e) {
      System.err.println(e.getMessage());
      e.printStackTrace(System.err);
    } finally {
      if (xProcess != null) {
        xProcess.destroy();
        xProcess = null;
      }
      if (logFileWriter != null) {
        logFileWriter.close();
        logFileWriter = null;
      }
      System.exit(0);
    }
  }

  private static final class ReaderWriterPipe
    implements Runnable
  {
    private BufferedReader reader;
    private PrintWriter pw;

    public ReaderWriterPipe(BufferedReader reader, PrintWriter pw)
    {
      this.reader = reader;
      this.pw = pw;
    }

    public void run()
    {
      try {
        while ((this.reader != null) && (this.pw != null)) {
          String sLine = this.reader.readLine();
          if (sLine == null) {
            break;
          }
          System.out.println(sLine);
          this.pw.println(sLine);
          this.pw.flush();
        }
      } catch (IOException e) {
        e.printStackTrace(System.err);
      } finally {
        if (this.reader != null) {
          try {
            this.reader.close();
          } catch (IOException e) {
            e.printStackTrace(System.err);
          }
          this.reader = null;
        }

        if (this.pw != null) {
          this.pw.close();
          this.pw = null;
        }
      }
    }
  }
}

package murex.rmi.loader;

import java.io.DataInputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintStream;
import java.net.URL;
import java.security.Permission;
import java.security.SecureClassLoader;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.Hashtable;
import java.util.Map;
import java.util.Properties;
import java.util.Vector;
import java.util.jar.JarFile;
import java.util.zip.ZipEntry;
import murex.rmi.loader.parser.CheckFile;
import murex.rmi.loader.parser.FileVersion;
import murex.rmi.loader.parser.Parser;
import murex.rmi.loader.parser.VersionParser;

public class FileServerClassLoader extends SecureClassLoader
{
  public static final String strJarFolder = "jar/";
  public static final String strBinFolder = "bin/";
  public static final String strFileVersion = "file.version";
  public static final String tagFiles = "Files";
  public static final String tagFile = "File";
  public static final String tagVersion = "Version";
  public static final String tagCheckSum = "CheckSum";
  private JarFile[] arrayJarFiles;
  private Hashtable hashtableMemory;

  FileServerClassLoader(URL url, URL urlFileServerSite)
    throws Exception
  {
    super(FileServerClassLoader.class.getClassLoader());
    String fileClassPath = FileServerGenericLoader.fileServletDownload();
    if (fileClassPath == null) {
      if ((url.getFile().trim().indexOf(".download") == -1) && (url.getFile().trim().indexOf(".classpath") == -1))
        throw new Exception(" no file .download is specified in your codebase.");
      fileClassPath = url.getFile();
    }

    String strAdaptedFilePath = fileClassPath;
    int iLastSeparatorIndex = fileClassPath.lastIndexOf(".");
    if (iLastSeparatorIndex != -1) {
      String strFilePathToAdapt = fileClassPath.substring(0, iLastSeparatorIndex);
      strFilePathToAdapt = strFilePathToAdapt.replace('.', '/');
      strAdaptedFilePath = strFilePathToAdapt + fileClassPath.substring(iLastSeparatorIndex);
    }
    fileClassPath = strAdaptedFilePath;

    createSubDirectories();

    byte[] jarListFile = FileServerGenericLoader.getFileFromFileServer(url, fileClassPath);
    String strJarListFile = new String(jarListFile);
    jarListFile = null;

    Parser parser = new Parser(strJarListFile);
    strJarListFile = null;
    CheckFile[] physical = parser.getPhysicalDownload();
    CheckFile[] memory = parser.getMemoryDownload();
    parser = null;

    if (RmiLoader.getMxProperties().get("murex.application.jni") == null)
    {
      URL physicalURL;
      URL physicalURL;
      if (urlFileServerSite != null)
        physicalURL = urlFileServerSite;
      else {
        physicalURL = url;
      }

      String strBinFileVersion = "bin/file.version";
      File fBinFileVersion = new File(strBinFileVersion);
      StringBuffer binStringBuffer = null;
      Map binFileVersion = null;
      if (fBinFileVersion.exists()) {
        System.out.println("Opening " + strBinFileVersion + "...");
        try {
          VersionParser versionParser = new VersionParser(fBinFileVersion.getPath());
          binFileVersion = versionParser.getFileVersion();
        } catch (Exception e) {
          e.printStackTrace();
        }
      }

      String strJarFileVersion = "jar/file.version";
      File fJarFileVersion = new File(strJarFileVersion);
      StringBuffer jarStringBuffer = null;
      Map jarFileVersion = null;
      if (fJarFileVersion.exists()) {
        System.out.println("Opening " + strJarFileVersion + "...");
        try {
          VersionParser versionParser = new VersionParser(fJarFileVersion.getPath());
          jarFileVersion = versionParser.getFileVersion();
        } catch (Exception e) {
          e.printStackTrace();
        }
      }

      File fLocalFileVersion = new File("file.version");
      StringBuffer localStringBuffer = null;
      Map localFileVersion = null;
      if (fLocalFileVersion.exists()) {
        System.out.println("Opening file.version...");
        try {
          VersionParser versionParser = new VersionParser(fLocalFileVersion.getPath());
          localFileVersion = versionParser.getFileVersion();
        } catch (Exception e) {
          e.printStackTrace();
        }

      }

      boolean bJFVdownload = false;
      boolean bBFVdownload = false;
      boolean bLFVdownload = false;

      System.out.println("Checking files...");
      ArrayList arrayJar = new ArrayList(physical.length);
      StringBuffer strCodebase = new StringBuffer("");
      for (int j = 0; j < physical.length; j++) {
        byte[] jarFile = null;
        String strFile = physical[j].getFile();
        String strDestFile = getDestFileName(strFile);
        boolean bGet = true;
        String strCheck = physical[j].getCheck();
        String strCheckSum = null;
        String strVersion = null;

        if ((strCheck != null) && (strCheck.equals("N"))) {
          bGet = !new File(strDestFile).exists();
          if (bGet) {
            strCheckSum = physical[j].getCheckSum();
          }
          else
          {
            Map map;
            Map map;
            if (isInJarFolder(strFile)) {
              map = jarFileVersion;
            }
            else
            {
              Map map;
              if (isInBinFolder(strFile))
                map = binFileVersion;
              else {
                map = localFileVersion;
              }
            }
            if (map != null) {
              FileVersion fv = (FileVersion)map.get(strFile);
              if ((fv != null) && 
                (fv.getCheckSum() != null)) {
                strCheckSum = fv.getCheckSum();
              }
            }

            if (strCheckSum == null) {
              strCheckSum = FileServerGenericLoader.getCheckSum(strFile);
            }
          }
        }
        else if ((strCheck != null) && (strCheck.equals("Y"))) {
          strCheckSum = physical[j].getCheckSum();
          if (!new File(strDestFile).exists()) {
            bGet = true;
          }
          else if (strCheckSum != null)
          {
            Map map;
            Map map;
            if (isInJarFolder(strFile)) {
              map = jarFileVersion;
            }
            else
            {
              Map map;
              if (isInBinFolder(strFile))
                map = binFileVersion;
              else {
                map = localFileVersion;
              }
            }
            String strCheckSum2 = null;
            if (map != null) {
              FileVersion fv = (FileVersion)map.get(strFile);
              if ((fv != null) && 
                (fv.getCheckSum() != null)) {
                strCheckSum2 = fv.getCheckSum();
              }
            }

            if (strCheckSum2 == null) {
              strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
            }
            bGet = !strCheckSum2.equals(strCheckSum);
          }

        }
        else if ((strCheck != null) && (strCheck.equals("A"))) {
          if (physical[j].getVersion() != null) {
            strVersion = physical[j].getVersion();
          }
          if (physical[j].getCheckSum() != null) {
            strCheckSum = physical[j].getCheckSum();
          }

          if (!new File(strDestFile).exists()) {
            bGet = true;
          }
          else
          {
            Map map;
            Map map;
            if (isInJarFolder(strFile)) {
              map = jarFileVersion;
            }
            else
            {
              Map map;
              if (isInBinFolder(strFile))
                map = binFileVersion;
              else {
                map = localFileVersion;
              }
            }
            if (map != null) {
              FileVersion fv = (FileVersion)map.get(strFile);
              if (fv != null) {
                if ((strVersion != null) && (fv.getVersion() != null)) {
                  bGet = !fv.getVersion().equals(strVersion);
                }
                else if (strCheckSum != null) {
                  if (fv.getCheckSum() != null) {
                    bGet = !fv.getCheckSum().equals(strCheckSum);
                  } else {
                    String strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
                    bGet = !strCheckSum2.equals(strCheckSum);
                  }
                }
              }
              else if (strCheckSum != null) {
                String strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
                bGet = !strCheckSum2.equals(strCheckSum);
              }

            }
            else if (strCheckSum != null) {
              String strCheckSum2 = FileServerGenericLoader.getCheckSum(strFile);
              bGet = !strCheckSum2.equals(strCheckSum);
            }
          }

        }

        if (bGet)
          try {
            jarFile = FileServerGenericLoader.getFileFromFileServer(physicalURL, strFile);
          } catch (Exception err1) {
            err1.printStackTrace();
            jarFile = null;
          }
        else {
          jarFile = new byte[0];
        }

        if (jarFile != null)
        {
          if (jarFile.length != 0) {
            FileOutputStream jarFileOutputStream = new FileOutputStream(strDestFile);
            jarFileOutputStream.write(jarFile);
            jarFileOutputStream.close();
            System.out.println(strDestFile + " copied.");
          }
          StringBuffer sb;
          StringBuffer sb;
          if (isInJarFolder(strFile)) {
            if (bGet)
            {
              bJFVdownload = true;
            }
            arrayJar.add(new JarFile(strDestFile));

            strCodebase.append("file:./");
            strCodebase.append(strDestFile);
            strCodebase.append(" ");

            if (jarStringBuffer == null) {
              jarStringBuffer = new StringBuffer();
              jarStringBuffer.append("<"); jarStringBuffer.append("Files"); jarStringBuffer.append(">\r\n");
            }
            sb = jarStringBuffer;
          }
          else
          {
            StringBuffer sb;
            if (isInBinFolder(strFile)) {
              if (bGet) {
                bBFVdownload = true;
              }
              if (binStringBuffer == null) {
                binStringBuffer = new StringBuffer();
                binStringBuffer.append("<"); binStringBuffer.append("Files"); binStringBuffer.append(">\r\n");
              }
              sb = binStringBuffer;
            }
            else {
              if (bGet)
              {
                bLFVdownload = true;
              }
              if (localStringBuffer == null) {
                localStringBuffer = new StringBuffer();
                localStringBuffer.append("<"); localStringBuffer.append("Files"); localStringBuffer.append(">\r\n");
              }
              sb = localStringBuffer;
            }
          }
          sb.append("<"); sb.append("File");
          if (strVersion != null) {
            sb.append(" ");
            sb.append("Version"); sb.append("=\""); sb.append(strVersion); sb.append("\"");
          }
          if (strCheckSum != null) {
            sb.append(" ");
            sb.append("CheckSum"); sb.append("=\""); sb.append(strCheckSum); sb.append("\"");
          }
          sb.append(">");
          sb.append(strFile);
          sb.append("</"); sb.append("File"); sb.append(">\r\n");
        }
      }
      if (!fBinFileVersion.exists()) {
        bBFVdownload = true;
      }
      if (!fJarFileVersion.exists()) {
        bJFVdownload = true;
      }
      if (!fLocalFileVersion.exists()) {
        bLFVdownload = true;
      }

      if ((binStringBuffer != null) && (bBFVdownload)) {
        binStringBuffer.append("</"); binStringBuffer.append("Files"); binStringBuffer.append(">\r\n");
        try {
          if (fBinFileVersion.exists()) {
            fBinFileVersion.delete();
          }
          FileOutputStream jarFileOutputStream = new FileOutputStream(strBinFileVersion);
          jarFileOutputStream.write(binStringBuffer.toString().getBytes());
          jarFileOutputStream.close();
          System.out.println(strBinFileVersion + " saved.");
        } catch (Exception e) {
          System.out.println(e.getMessage());
        }
      }

      if ((jarStringBuffer != null) && (bJFVdownload)) {
        jarStringBuffer.append("</"); jarStringBuffer.append("Files"); jarStringBuffer.append(">\r\n");
        try {
          if (fJarFileVersion.exists()) {
            fJarFileVersion.delete();
          }
          FileOutputStream jarFileOutputStream = new FileOutputStream(strJarFileVersion);
          jarFileOutputStream.write(jarStringBuffer.toString().getBytes());
          jarFileOutputStream.close();
          System.out.println(strJarFileVersion + " saved.");
        } catch (Exception e) {
          System.out.println(e.getMessage());
        }
      }

      if ((localStringBuffer != null) && (bLFVdownload)) {
        localStringBuffer.append("</"); localStringBuffer.append("Files"); localStringBuffer.append(">\r\n");
        try {
          if (fLocalFileVersion.exists()) {
            fLocalFileVersion.delete();
          }
          FileOutputStream jarFileOutputStream = new FileOutputStream("file.version");
          jarFileOutputStream.write(localStringBuffer.toString().getBytes());
          jarFileOutputStream.close();
          System.out.println("file.version saved.");
        } catch (Exception e) {
          System.out.println(e.getMessage());
        }
      }

      this.arrayJarFiles = ((JarFile[])arrayJar.toArray(new JarFile[0]));

      System.getProperties().put("java.rmi.server.codebase", strCodebase.toString());
      this.hashtableMemory = new Hashtable();
      for (int j = 0; j < memory.length; j++) {
        String strFile = memory[j].getFile();
        byte[] classFile = null;
        try {
          classFile = FileServerGenericLoader.getFileFromFileServer(url, strFile);
        } catch (Exception err1) {
          classFile = null;
        }
        if (classFile != null)
          this.hashtableMemory.put(strFile, classFile);
      }
    }
    else {
      StringBuffer strCodebase = new StringBuffer("");
      JarFile[] jarFile = new JarFile[physical.length];
      int iCount = 0;
      for (int j = 0; j < physical.length; j++)
        try {
          String strFile = physical[j].getFile();
          if (isInJarFolder(strFile)) {
            String strDestFile = getDestFileName(strFile);
            jarFile[j] = new JarFile(strDestFile);
            iCount++;
            strCodebase.append("file:./");
            strCodebase.append(strDestFile);
            strCodebase.append(" ");
          }
        }
        catch (Exception e) {
        }
      System.getProperties().put("java.rmi.server.codebase", strCodebase.toString());

      this.arrayJarFiles = new JarFile[iCount];
      int j = 0;
      for (int i = 0; i < jarFile.length; i++) {
        if (jarFile[i] != null) {
          this.arrayJarFiles[j] = jarFile[i];
          j++;
        }
      }
      jarFile = null;
      strCodebase = null;

      this.hashtableMemory = new Hashtable();
      for (j = 0; j < memory.length; j++) {
        String strFile = memory[j].getFile();
        byte[] classFile = null;
        try {
          classFile = FileServerGenericLoader.getFileFromFileServer(url, strFile);
        } catch (Exception err1) {
          classFile = null;
        }
        if (classFile != null)
          this.hashtableMemory.put(strFile, classFile);
      }
    }
    physical = null;
    memory = null;
  }

  public Class findClass(String name) throws ClassNotFoundException {
    Class classData = null;
    byte[] b = loadClassData(name);
    if (b == null)
      throw new ClassNotFoundException(name);
    int index = name.lastIndexOf('.');
    if (index != -1) {
      String pkgname = name.substring(0, index);
      Package pkg = getPackage(pkgname);
      if (pkg == null) {
        definePackage(pkgname, null, null, null, null, null, null, null);
      }
    }
    classData = defineClass(name, b, 0, b.length);
    return classData;
  }

  private byte[] loadClassData(String name) {
    byte[] bytecodes = null;
    name = name.replace('.', '/') + ".class";
    for (int i = 0; (i < this.arrayJarFiles.length) && (bytecodes == null); i++)
      try {
        ZipEntry zipEntry = this.arrayJarFiles[i].getEntry(name);
        if (zipEntry != null) {
          InputStream in1 = this.arrayJarFiles[i].getInputStream(zipEntry);
          DataInputStream in2 = new DataInputStream(in1);
          bytecodes = new byte[(int)zipEntry.getSize()];
          in2.readFully(bytecodes);
          try {
            in2.close();
          } catch (Exception ee) {
          }
        }
      }
      catch (Exception e) {
      }
    if (bytecodes == null) {
      bytecodes = (byte[])this.hashtableMemory.get(name);
    }
    return bytecodes;
  }

  protected Enumeration findResources(String name)
    throws IOException
  {
    Vector v = null;
    for (int i = 0; i < this.arrayJarFiles.length; i++) {
      try {
        ZipEntry zipEntry = this.arrayJarFiles[i].getEntry(name);
        if (zipEntry != null) {
          if (v == null) {
            v = new Vector();
          }
          v.add(new URL("jar:file:./" + getDestFileName(this.arrayJarFiles[i].getName()) + "!/" + name));
        }
      } catch (Exception e) {
        if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
          e.printStackTrace();
        }
      }
    }
    if (v != null) {
      return v.elements();
    }
    return null;
  }

  protected URL findResource(String name)
  {
    URL url = null;
    for (int i = 0; (i < this.arrayJarFiles.length) && (url == null); i++) {
      try {
        ZipEntry zipEntry = this.arrayJarFiles[i].getEntry(name);
        if (zipEntry != null)
          url = new URL("jar:file:./" + getDestFileName(this.arrayJarFiles[i].getName()) + "!/" + name);
      }
      catch (Exception e) {
        if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
          e.printStackTrace();
        }
      }
    }
    return url;
  }

  public static String getDestFileName(String strFile)
  {
    String strDestFile;
    String strDestFile;
    if (isInBinFolder(strFile)) {
      strDestFile = "bin/" + getFileName(strFile);
    }
    else
    {
      String strDestFile;
      if (isInJarFolder(strFile))
        strDestFile = "jar/" + getFileName(strFile);
      else
        strDestFile = getFileName(strFile);
    }
    return strDestFile;
  }

  public static boolean isInJarFolder(String strFile) {
    return strFile.endsWith("jar");
  }

  public static boolean isInBinFolder(String strFile) {
    return (strFile.endsWith(".dll")) || (strFile.endsWith(".so")) || (strFile.endsWith(".a")) || (strFile.endsWith(".sl")) || (strFile.endsWith(".exe")) || (strFile.endsWith(".tlb"));
  }

  private static void createSubDirectories() {
    File fileJarFolder = new File("jar/");
    if (!fileJarFolder.exists()) {
      if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
        System.out.println("Creating folder jar/");
      }
      fileJarFolder.mkdir();
    }

    File fileBinFolder = new File("bin/");
    if (!fileBinFolder.exists()) {
      if (RmiLoader.getMxProperties().get("murex.application.jni") == null) {
        System.out.println("Creating folder bin/");
      }
      fileBinFolder.mkdir();
    }
  }

  private static String getFileName(String strPath) {
    String strFileName = strPath;
    int iSeparatorIndex = strPath.lastIndexOf("/");
    if (iSeparatorIndex != -1) {
      strFileName = strPath.substring(iSeparatorIndex + 1);
    }
    iSeparatorIndex = strPath.lastIndexOf("\\");
    if (iSeparatorIndex != -1) {
      strFileName = strPath.substring(iSeparatorIndex + 1);
    }
    return strFileName;
  }

  private class ClassLoaderSecurityManager extends SecurityManager
  {
    private ClassLoaderSecurityManager()
    {
    }

    public void checkPermission(Permission perm)
    {
    }

    public void checkPermission(Permission perm, Object context)
    {
    }
  }
}

package murex.rmi.loader;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.Authenticator;
import java.net.PasswordAuthentication;
import java.net.URL;
import java.net.URLConnection;
import java.util.Enumeration;
import java.util.Hashtable;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

public class FileServerGenericLoader
{
  public static final String tagFile = "File";
  public static final String tagVersion = "Version";
  public static final String tagCheckSum = "CheckSum";
  public static final String tagException = "EXCEPTION";
  public static final String tagMurexAnswer = "Murex-Answer";
  public static final int nbFile = 30;

  public static byte[] getFileFromFileServer(URL url, String strFile, boolean doCheck, String strVersion, String strCheckSum)
    throws Exception
  {
    if (strFile.charAt(0) != '/') {
      strFile = "/" + strFile;
    }

    String strParameter = "";
    if (doCheck) {
      if (strVersion != null) {
        strParameter = "Version=" + strVersion;
      }
      if (strCheckSum != null) {
        if (strVersion != null) {
          strParameter = strParameter + "&";
        }
        strParameter = strParameter + "CheckSum=" + strCheckSum;
      }

      if ((strVersion == null) && (strCheckSum == null)) {
        strCheckSum = getCheckSum(strFile);
        if (strCheckSum != null) {
          strParameter = "CheckSum=" + strCheckSum;
        }
      }
    }

    if (!strParameter.equals("")) {
      strParameter = "&" + strParameter;
    }
    strParameter = "InternalVersion=1" + strParameter;
    String strRequest;
    if (fileServletDownload() != null) {
      String strRequest = url.getFile();
      strRequest = strRequest + "?File=" + strFile;
      if (!strParameter.equals(""))
        strRequest = strRequest + "&" + strParameter;
    }
    else {
      strRequest = strFile;
      if (!strParameter.equals("")) {
        strRequest = strRequest + "?" + strParameter;
      }
    }

    URL urlToFileServer = new URL(url.getProtocol(), url.getHost(), url.getPort(), strRequest);
    URLConnection urlConnection = urlToFileServer.openConnection();
    urlConnection.setUseCaches(false);
    urlConnection.setRequestProperty("MUREX-EXTENSION", "murex/murex.rmi.loader.FileServerGenericLoader");
    InputStream inputStream = urlConnection.getInputStream();

    if (urlConnection.getContentLength() == -1) {
      throw new Exception("unable to get file " + strFile + " from " + url + ".");
    }

    String strStatus = urlConnection.getHeaderField("Murex-Answer");
    if ((strStatus != null) && (strStatus.equals("EXCEPTION"))) {
      byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
      throw new Exception(new String(inputFile));
    }

    byte[] inputFile = getBodyStream(inputStream, urlConnection.getContentLength());
    inputStream.close();
    return inputFile;
  }

  public static byte[] getFileFromFileServer(URL url, String strFile) throws Exception {
    return getFileFromFileServer(url, strFile, false, null, null);
  }

  public static String fileServletDownload() {
    return (String)RmiLoader.getMxProperties().get("/MXJ_SERVLET_DOWNLOAD:");
  }

  public static String getCheckSum(String strFile) {
    String strCheckSum = null;
    if (FileServerClassLoader.isInJarFolder(strFile))
      try {
        String strDestFile = FileServerClassLoader.getDestFileName(strFile);
        JarFile jarFile = new JarFile(strDestFile);
        Enumeration enum = jarFile.entries();
        StringBuffer result = new StringBuffer();
        int nb = 0;
        long sum = 0L;
        while (enum.hasMoreElements()) {
          JarEntry jarEntry = (JarEntry)enum.nextElement();
          if (nb < 30) {
            sum += jarEntry.getSize();
          } else {
            result.append(sum);
            sum = jarEntry.getSize();
            nb = 0;
          }
        }
        if (sum != 0L) {
          result.append(sum);
        }
        jarFile.close();
        strCheckSum = result.toString();
      }
      catch (Exception e) {
      }
    else try {
        File file = new File(FileServerClassLoader.getDestFileName(strFile));
        strCheckSum = file.length() + "";
      }
      catch (Exception e)
      {
      } return strCheckSum;
  }

  public static byte[] getBodyStream(InputStream inputStream, int count) {
    byte[] tab = new byte[count];
    try {
      int iReceived = 0;
      int iTempReceived = 0;
      while ((iTempReceived = inputStream.read(tab, iReceived, count - iReceived)) > 0)
        iReceived += iTempReceived;
    }
    catch (IOException e) {
    }
    return tab;
  }

  static
  {
    if ((System.getProperty("murex.proxy.username") != null) && (System.getProperty("murex.proxy.password") != null))
      Authenticator.setDefault(new PasswordAuthenticator(System.getProperty("murex.proxy.username"), System.getProperty("murex.proxy.password"))); 
  }

  private static class PasswordAuthenticator extends Authenticator {
    private String strUsername;
    private String strPassword;
    private PasswordAuthentication passwordAuthentication;

    public PasswordAuthenticator(String strUsername, String strPassword) { this.passwordAuthentication = new PasswordAuthentication(strUsername, strPassword.toCharArray()); }

    protected PasswordAuthentication getPasswordAuthentication()
    {
      return this.passwordAuthentication;
    }
  }
}

package murex.rmi.loader;

import java.net.URL;
import java.rmi.RMISecurityManager;
import java.security.Permission;
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Map;
import java.util.Properties;
import java.util.StringTokenizer;

public class RmiLoader
{
  public static final String MXJ_ARGS_CLASS_NAME = "/MXJ_CLASS_NAME:";
  public static final String MXJ_ARGS_FILE_SERVER_REMOTE_SITE_URL = "/MXJ_FILE_SERVER_REMOTE_SITE_URL:";
  public static final String propertyFILE_SERVER_REMOTE_SITE_URL = "murex.application.codebase.remote.site";
  public static final String MXJ_ARGS_SERVLET_DOWNLOAD = "/MXJ_SERVLET_DOWNLOAD:";
  protected static Hashtable mxProperties = new Hashtable();
  protected URL urlJarFile;
  protected URL urlFileServerSite;
  protected String strClientClassName;
  protected FileServerClassLoader fileServerClassLoader;

  protected RmiLoader(String strURL, String strClientClassName)
    throws Exception
  {
    System.setSecurityManager(new RMIClientBootstrapSecurityManager(null));
    this.strClientClassName = strClientClassName;
    int index = strURL.indexOf("?");
    if (index != -1) {
      Map mapParameter = new HashMap();
      String strQuery = strURL.substring(index + 1);
      StringTokenizer stk = new StringTokenizer(strQuery, "&");
      while (stk.hasMoreElements()) {
        String str = (String)stk.nextElement();
        String strName = str.substring(0, str.indexOf("="));
        String strValue = str.substring(str.indexOf("=") + 1);
        mapParameter.put(strName.trim(), strValue.trim());
      }
      strURL = strURL.substring(0, index);
      mxProperties.put("/MXJ_SERVLET_DOWNLOAD:", mapParameter.get("File"));
    }
    initURL(strURL);
  }

  protected RmiLoader(String[] strArgs) throws Exception {
    System.setSecurityManager(new RMIClientBootstrapSecurityManager(null));
    for (int i = 0; i < strArgs.length; i++) {
      if (strArgs[i].startsWith("/MXJ_CLASS_NAME:"))
        this.strClientClassName = strArgs[i].substring("/MXJ_CLASS_NAME:".length());
      else if (strArgs[i].startsWith("/MXJ_FILE_SERVER_REMOTE_SITE_URL:"))
        this.urlFileServerSite = new URL(strArgs[i].substring("/MXJ_FILE_SERVER_REMOTE_SITE_URL:".length()));
      else if (strArgs[i].startsWith("/MXJ_SERVLET_DOWNLOAD:")) {
        mxProperties.put("/MXJ_SERVLET_DOWNLOAD:", strArgs[i].substring("/MXJ_SERVLET_DOWNLOAD:".length()));
      }
    }
    mxProperties.put("murex.rmi.loader.RmiLoader.arguments", strArgs);
    Properties p = System.getProperties();
    String strURL = p.getProperty("murex.application.codebase") == null ? p.getProperty("java.rmi.server.codebase") : p.getProperty("murex.application.codebase");
    initURL(strURL);
  }

  private void initURL(String strURL) throws Exception {
    if (strURL == null)
      throw new Exception(" no codebase specified.");
    mxProperties.put("murex.application.codebase", strURL);
    this.urlJarFile = new URL(strURL);
  }

  public static Hashtable getMxProperties() {
    return mxProperties;
  }

  public Object startRunning() throws Exception {
    if ((this.strClientClassName == null) || (this.strClientClassName.equals(""))) {
      throw new Exception("Class name not found");
    }

    if ((mxProperties.get("/MXJ_FILE_SERVER_REMOTE_SITE_URL:") != null) && (this.urlFileServerSite == null)) {
      this.urlFileServerSite = new URL((String)mxProperties.remove("/MXJ_FILE_SERVER_REMOTE_SITE_URL:"));
    }

    if (this.urlFileServerSite == null) {
      Properties p = System.getProperties();
      String strFileServerSite = (String)p.get("murex.application.codebase.remote.site");
      if (strFileServerSite != null) {
        this.urlFileServerSite = new URL(strFileServerSite);
      }
    }
    if (this.urlFileServerSite != null) {
      mxProperties.put("murex.application.codebase.remote.site", this.urlFileServerSite.toString());
    }

    this.fileServerClassLoader = new FileServerClassLoader(this.urlJarFile, this.urlFileServerSite);
    Thread.currentThread().setContextClassLoader(this.fileServerClassLoader);
    Class clientClass = Class.forName(this.strClientClassName, true, this.fileServerClassLoader);

    Runnable client = (Runnable)clientClass.newInstance();
    client.run();
    return client;
  }

  public Class findClass(String strClassName) throws Exception {
    return this.fileServerClassLoader.loadClass(strClassName);
  }

  public static void main(String[] strArgs)
  {
    try
    {
      RmiLoader rmiLoader = new RmiLoader(strArgs);
      rmiLoader.startRunning();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  private class RMIClientBootstrapSecurityManager extends RMISecurityManager
  {
    private RMIClientBootstrapSecurityManager()
    {
    }

    public synchronized void checkCreateClassLoader()
    {
    }

    public synchronized void checkConnect(String host, int port)
    {
    }

    public synchronized void checkAccess(Thread t)
    {
    }

    public synchronized void checkAccess(ThreadGroup g)
    {
    }

    public synchronized void checkPropertiesAccess()
    {
    }

    public void checkPermission(Permission perm)
    {
    }

    public void checkPermission(Permission perm, Object context)
    {
    }

    RMIClientBootstrapSecurityManager(RmiLoader.1 x1)
    {
      this();
    }
  }
}

package murex.rmi.loader;

import java.io.PrintStream;

public class Version
{
  public static final String strVersion = "1";
  public static final String tagInternalVersion = "InternalVersion";

  public static void printVersion()
  {
    System.out.println("The version of this class loader is 1.");
    System.out.println("It downloads files which extensions are .download.");
    System.out.println("This kind of files contains the list of files to be downloaded with its checksums and versions (if declared in the fileserver).");
    System.out.println("It can download this file from a file server or from a servlet.");
    System.out.println("");
    System.out.println("The downloading of a given file is decided by checking the Check attribute as follows : ");
    System.out.println("");
    System.out.println("    1-The attribute Check is set to N (No) :");
    System.out.println("    -------------------------------------------");
    System.out.println("    if the file exists nothing is done, if not the file is downloaded.");
    System.out.println("");
    System.out.println("    2-The attribute Check is set to Y (Yes) :");
    System.out.println("    -------------------------------------------");
    System.out.println("    if the file exists and the checksum is equal to the one given by the fileserver nothing is done, if not the file is downloaded.");
    System.out.println("");
    System.out.println("    3-The attribute Check is set to A (Automatic) :");
    System.out.println("    -------------------------------------------");
    System.out.println("    if the file has a version and is the same than the one given by the fileserver nothing is done, if not step 2 is done.");
    System.out.println("");
    System.out.println("At the end, versions and checksums are saved locally in files file.version.");
  }

  public static void main(String[] argv) {
    printVersion();
  }
}

package murex.xml.server.mx.mxnative.mxloader;

import java.io.PrintStream;
import java.util.Hashtable;
import murex.rmi.loader.RmiLoader;

public class RmiMxLoader extends RmiLoader
{
  private static PrintStream printStream = null;

  protected RmiMxLoader(String strURL, String strClientClassName)
    throws Exception
  {
    super(strURL, strClientClassName);
  }

  public static RmiMxLoader create(String strURL) {
    RmiMxLoader rmiMxLoader = null;
    try {
      rmiMxLoader = new RmiMxLoader(strURL, "murex.xml.server.mx.mxnative.MxHome");
      mxProperties.put("murex.application.jni", "true");
    } catch (Exception e) {
      printStackTrace(e);
    }
    return rmiMxLoader;
  }

  public static RmiMxLoader create(String strURL, String strHomeClass) {
    RmiMxLoader rmiMxLoader = null;
    try {
      rmiMxLoader = new RmiMxLoader(strURL, strHomeClass);
    } catch (Exception e) {
      printStackTrace(e);
    }
    return rmiMxLoader;
  }

  public static RmiMxLoader create(String strURL, String strHomeClass, String strJniMode) {
    RmiMxLoader rmiMxLoader = null;
    try {
      rmiMxLoader = new RmiMxLoader(strURL, strHomeClass);
      if ((strJniMode != null) && (strJniMode.equals("yes")))
        mxProperties.put("murex.application.jni", "true");
    }
    catch (Exception e) {
      printStackTrace(e);
    }
    return rmiMxLoader;
  }

  public void pushArguments(String strCode, String strValue) {
    mxProperties.put(strValue, strCode);
  }

  public Object commitArguments() {
    Object obj = null;
    try {
      obj = startRunning();
    } catch (Exception e) {
      printStackTrace(e);
    }
    return obj;
  }

  public Class findClass(String strClassName) {
    Class c = null;
    try {
      c = super.findClass(strClassName);
    } catch (Exception e) {
      printStackTrace(e);
    }
    return c;
  }

  public static void printStackTrace(Throwable e)
  {
    if (printStream != null)
      e.printStackTrace(printStream);
  }

  public static void println(String str)
  {
    if (printStream != null)
      printStream.println(str);
  }
}


## All Murex 2.1 packages (from  clease case server) 
[1477277@dl1101 ~]$ bash
[1477277@dl1101 ~]$ cd /
[1477277@dl1101 /]$ ls
a     b         bin   c       clearcase  dev_vob      etc   lib    lost+found  mnt  opt  proc  rel_vob  sbin     shared  sys  usr  view
apps  bhanu.sh  boot  cgroup  dev        dev_vob_new  home  lib64  media       nsr  pfs  puma  root     selinux  srv     tmp  var  vobs
[1477277@dl1101 /]$ cd dev_vob
[1477277@dl1101 dev_vob]$ ls
common  packages
[1477277@dl1101 dev_vob]$ cd packages
[1477277@dl1101 packages]$ ls
blizzard  castor  enconnect    mars  pollux    preprocessor  scrittura2_new  scritturamis   vast2.0
borvo2    dcrm    leps_script  nike  poseidon  s2bfx         scrittura2_prj  syndicatebook  wmb
[1477277@dl1101 packages]$ cd poseidon/
[1477277@dl1101 poseidon]$ ls
[1477277@dl1101 poseidon]$ ct setview 1477277
[1477277@dl1101 poseidon]$ ls
bo               dm_audit_exts    dm_pl_ext       dpstools        housekeeping      medusa              mx_api               pandora         regression
cadex            dm_basel         dm_pl_fdr       ebbs            install-me.sh     merlin              mx_api_x86           pay_db_objects  sabre_ird_fxo
calbal           dm_cash_flow     dm_pmnt_fdr     ebs             irec              mktrates            mx_api_x86_var       pemtools        SolarisStudioX86
callcontra       dm_complvar_fdr  dmpv01          endymion        itrs              mls                 mxml                 perseus         tools
caxton           dm_com_sim       dm_reg_exts     eod             itrstools         model               mxtools              plutus          toyota
common           dm_cs_ext        dm_sentry       eod_auto        jobdefs           monitor             NAG                  plutus2         triton
common_x86       dm_cs_fdr        dm_sim_fdr      Extractions     launchers         murex               ndf                  pos_calrecon    tt.txt
crt              dm_fdr_sens      dm_static_exts  fabs            lost+found        murexg2000          nike                 pos_mxisft      varcon
data_conversion  dm_maketer_ext   dm_static_fdr   fdr_db_objects  Makefilebuild.sh  murex_housekeeping  objectsrelease_info  primetrade      varex
datamart         dm_mv_ext        dm_sv_fdr       Feeders         markit            murex_lib           old_code             prorisk         varutils
dm_acct_fdr      dm_mv_fdr        dm_test         fm_tools        mats              murfi               olink                proteus         web
[1477277@dl1101 poseidon]$


## application SERVER
						
在B/S模式中，服务器端用servlet来为web客户的请求提供服务。Servlet服务扩展了web服务器的功能，即由web服务器接收和处理客户请求，然后把请求传给servlet,并把servlet处理的结果返回给客户。						
servlet 容器与servlet之间的接口是由java.servlet.api定义的，在此api中定义了servlet的各种方法，这些方法在servlet生命周期的不同阶段被servlet容器调用，tomcat服务器由一系列可配置的组件构成，tomcat组件可以在conf/server.xml文件当中进行配置。组件包括：						
						
<Server> => 代表整个servelet容器，是tomcat实例的顶层元素。						
      <Service>						
			<Connector>			
			     客户与服务器之间的通信接口			
			</Connector>			
			<Connector>			
			</Connector>			
			<Engine> => 处理在同一个<Service>中所有<Connector>元素接受到的用户请求			
				<Host>  => 每个<Host>元素定义了一个虚拟主机，它可以包含一个或多个web应用。		
					<Context path=''> 	
					     为特定的web应用处理所有用户请求，每个<Context>元素代表了运行在虚拟机上的单个web应用。	
						 每个web应用有唯一的Context，当java web应用运行时，Servlet容器为每个web应用创建唯一的ServletContext对象，它被整个web应用中所有的组件共享
					</Context>	
					...	
					<Context> 	
					</Context>	
				</Host>		
			</Engine>			
      </Service>						
	  					
	  <Service>					
	  </Service>					
</Server>						
						
						
Murex采用C/S结构实现的，所以Server端没有Tomcat服务器，需要我们自己用TCP/IP socket来实现类似于TOMCAT的功能。						
类似一个应用服务器中可以部署多个应用程序，我们的服务器端也可以配置和部署多个Murex应用，以为团队提供多个测试环境。						
						
Murex Clinet 先通过RMI技术发送请求给fileServer as All Resource Files all are deployed on file server.. 						
Once File server received request it will load sites.mxres (./fs/public/mxres/sites/) to fetch the XML server & hub host & port 						
						
XML server就相当于Tomcat服务器，顾名思义使用XML技术接收和处理客户请求，同时负责Services的 registration 和 lifecycle management (service’s creation, checking, termination etc..)						
hub server 相当于一个虚拟机？						
						
Declare, customize and launch different type of services						
Existing in File server as e.g. launcherall.mxres under ./fs/public/mxres/common/						
./launchmxj.app –l / -s / -k						
Service processes are located on the host of the launcher						
						
The features of Murex can work only when the corresponding services are running on the service host.						
						
						
						
						
						
						
						
						
						
magapp15a:/shared/home/murex$ less  .profile　　　或　　　　magapp15a:/shared/home/posop$ less  .profile						
	# Control-M related alias					
	alias jobdefs='cd /shared/opt/SCB/pos_eod/live/config/jobdefs'					
	alias ctm='cd /shared/opt/SCB/pos_eod/live'					
	alias varex='cd /shared/opt/SCB/pos_varex/live'					
	alias varu='cd /shared/opt/SCB/pos_varutils/live'					
	alias dev1utils='cd /shared/opt/pos/mxg/mxg_bodev1/scb/utils'					
	alias sgutils='cd /shared/opt/pos/mxg/mxg_bodev1/scb/utils'					
	alias dev2utils='cd /shared/opt/pos/mxg/mxg_bodev2/scb/utils'					
	alias dailyutils='cd /shared/opt/pos/mxg/mxg_bau_daily/scb/utils'					
	alias dev1rep='cd /shared/opt/pos/mxg/mxg_bodev1/scb/reports'					
	alias dev2rep='cd /shared/opt/pos/mxg/mxg_bodev2/scb/reports'					
	alias dailyrep='cd /shared/opt/pos/mxg/mxg_bau_daily/scb/reports'					
	alias l='ls -lrt'					
						
"
"						
magapp15a:/shared/home/murex$ less .env_alias						
	alias mxghome='cd /shared/opt/pos/mxg'					
						
	# Environment and POS Market Rates aliases ( 2006 )					
	# -- Murex related aliases (2006)					
	alias cdd='cd fs/public/mxres/common/dbconfig'					
	alias cdl='cd fs/public/mxres/common'					
	alias cds='cd fs/public/mxres/sites'					
						
						
						
						
						
						
						
						
						
						
						
						
						
						
http://uklpadinf01a.uk.standardchartered.com/ganglia/?c=MUREX%20G2000&m=load_one&r=hour&s=by%20name&hc=4&mc=2						
						
						
						
						
						
						
						
						
	Restart Server: must use murex not posop					
						
						
	magapp14a:/shared/opt/pos/mxg/mxg_com_daily$ ../mxg_stop_services.sh					
						
	magapp14a:/shared/opt/pos/mxg/mxg_eod$ ../mxg_start_services.sh Y YY Y Y &					
						
						
	launchmxj.app -s					
						
						
						
	MXG_FDR :-					
	Login: magapp15a: as murex					
	cd /shared/mrxdev_pos/mxg_fdr/live					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr stop					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr start					
						
						
						
						
						
						
						
mx						
	/shared/opt/SCB/dev_launchers/binaries/live/mx					


## MUREX CLIENT
On Windows Operation System Murex Client is a .bat file as shown below		
########################################		
@ECHO OFF		
		
REM Mx G2000 Client Launcher		
REM Mofify this script to match your java and server environnement		
REM For 2.2.8 and 2.2.9		
REM V2.3		
		
setlocal		
		
cd %TEMP%		
		
echo %TEMP%		
		
if exist mxjboot.jar goto create		
		
echo open gmsitnfs.uk.standardchartered.com>>getboot.cmd		
echo posop>>getboot.cmd		
echo posop123>>getboot.cmd		
echo bin>>getboot.cmd		
echo cd /shared/opt/SCB/pos_murex/live/utils/jar>>getboot.cmd		
echo cd /shared/home/murex/MxSCBTools/tomcat/webapps/mxmlmonit/clients>>getboot.cmd		
echo get mxjboot.jar>>getboot.cmd		
echo quit>>getboot.cmd		
ftp -s:getboot.cmd		
del getboot.cmd		
		
:create		
		
IF EXIST "C:\Progra~1\MXG2000\J2RE1.4.2_08" (		
SET JAVAHOME="C:\Progra~1\MXG2000\J2RE1.4.2_08"		
) ELSE (		
SET JAVAHOME="C:\Progra~1\Java\j2re1.4.2_08"		
)		
		
SET MXJ_FILESERVER_HOST=magapp15a.uk.standardchartered.com		
SET MXJ_FILESERVER_PORT=21201		
SET MXJ_SITE_NAME=default		
SET MXJ_DESTINATION_SITE_NAME=mxg2k_fdr_live		
SET MXJ_PLATFORM_NAME=MX		
SET MXJ_PROCESS_NICK_NAME=MXG2K_FDR_LIVE		
		
		
SET PATH=%JAVAHOME%\jre\bin;%JAVAHOME%\jre\bin\classic;%JAVAHOME%\bin;%JAVAHOME%\bin\classic;%PATH%		
		
SET PATH=%PATH%;bin		
SET MXJ_JAR_FILELIST=murex.download.guiclient.download		
SET MXJ_POLICY=java.policy		
SET MXJ_BOOT=mxjboot.jar		
SET MXJ_CONFIG_FILE=client.xml		
		
IF EXIST jar\%MXJ_BOOT% copy jar\%MXJ_BOOT% . >NUL		
		
title %~n0 FS:%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST%  Xml:%MXJ_SITE_NAME% /PLATF:%MXJ_PLATFORM_NAME% /NNAME:%MXJ_PROCESS_NICK_NAME%		
		
java -Xmx512m -cp %MXJ_BOOT% -Djava.security.policy=%MXJ_POLICY% -Djava.rmi.server.codebase=http://%MXJ_FILESERVER_HOST%:%MXJ_FILESERVER_PORT%/%MXJ_JAR_FILELIST% murex.rmi.loader.RmiLoader /MXJ_SITE_NAME:%MXJ_SITE_NAME% /MXJ_DESTINATION_SITE_NAME:%MXJ_DESTINATION_SITE_NAME% /MXJ_CLASS_NAME:murex.gui.xml.XmlGuiClientBoot /MXJ_PLATFORM_NAME:%MXJ_PLATFORM_NAME% /MXJ_PROCESS_NICK_NAME:%MXJ_PROCESS_NICK_NAME% /MXJ_CONFIG_FILE:%MXJ_CONFIG_FILE% %1 %2 %3 %4 %5 %6		
		
title Command Prompt		
endlocal		
pause		
########################################		
解析：该批处理程序是通过java命令行启动客户端的murex.rmi.loader.RmiLoader 程序		
		
Java命令行基本结构：		
java [ options ] class [ argument ... ] 或 java [ options ] -jar file.jar [ argument ... ]		
		
说明：		
（1）options ：命令行选项。启动器有一组标准选项，当前的运行时环境支持这些选项并且将来的版本也将支持它们。还有一组其它的非标准选项是特定于目前的虚拟机实现的，将来可能要有变化。非标准选项以 -X 打头。		
     #标准选项：		
     .-cp 类路径,指定一个用于查找类文件的列表，它由目录、 JAR 归档文件和 ZIP 归档文件组成。类路径项用分号 (;) 分隔。指定 -classpath 或 -cp 将覆盖 CLASSPATH		
     .-D属性=值,设置系统属性的值。		
     .-jar JAR归档文件的名称，注意：JAR 归档文件的名称不是启动类的类名。启动类由 Main-Class 清单头指定。JAR 文件是所有用户类的源，其它的用户类路径设置将被忽略。		
     .-verbosee或-verbose:class,显示每个所加载的类的信息。		
     .-verbose:gc ,报告每个垃圾收集事件。 		
     .-verbose:jni ,报告有关本地方法的使用和其它 Java 平台相关代码接口活动的信息。		
     .-version ,显示版本信息并退出。 		
     .-? 或-help ,显示用法信息并退出。 		
     .-X ,显示非标准选项的有关信息并退出。 		
     #非标准选项：		
     .-Xbootclasspath:自举类路径 ,指定以分号分隔的目录、 JAR 归档文件和 ZIP 归档文件列表，用以查找自举类文件。这些自举类文件用来取代 JDK 1.2 软件中所包括的自举类文件。 		
     .-Xdebug ,启动激活的调试器。Java 解释器将输出一密码供 jdb 使用。有关详细资料及程序示例，请参阅 jdb 说明。 		
     .-Xnoclassgc ,禁用类垃圾收集 		
     .-Xms$V ,指定内存分配池的初始容量$V。该值$V必须大于 1000。要使该值扩大 1000 倍，须附加上字母 k，要使该值扩大一百万倍，须附加上字母 m。缺省值为 1m。 		
     .-Xmx$v ,指定内存分配池的最大容量$V。该值$V必须大于 1000。要将它扩大 1000 倍，须附加上字母 k，要将该值扩大一百万倍，须附加上字母 m。缺省值为 16m。 		
     .-Xrunhprof[:help][:<子选项>=<值>,...] ,启用 cpu 、堆或监视器监控操作。该选项后面一般跟着一个列表，该列表由以逗号分隔的 "<子选项>=<值>" 对所组成。运行命令 java -Xrunhprof:help 可获得子选项及其缺省值的列表。 		
     .-Xrs ,减少操作系统信号的使用。 		
     .-Xcheck:jni ,对 Java 平台相关代码接口函数进行额外检查。 		
		
（2）class ：要调用的类名。 或 file.jar ：要调用的 jar 文件名。只与 -jar 一起使用。		
     缺省情况下，第一个非选项参数是要调用的类名。应当使用全限定类名。如果指定了 -jar 选项，那么第一个非选项参数是 JAR 归档文件的名称，该归档文件包含应用程序的类和资源文件以及 Main-Class 清单头指定的启动类。 		
（3）argument ：传给 main 函数的参数。 类名或 JAR 文件名后的非选项参数被传递给 main 函数。		
（4）%MXJ_PLATFORM_NAME%: 批处理中定义的变量		
%1 %2 %3 %4 %5 %6：表示参数，参数是指在运行批处理文件时在文件名后加的以空格（或者Tab）分隔的字符串。变量可以从%0到%9，%0表示批处理命令本身，其它参数字符串用%1到%9顺序表示。%~dp0：表示批处理所在目录。		
例1：C:根目录下有一批处理文件名为f.bat，内容为：@echo offformat %1如果执行C:\>f a:那么在执行f.bat时，%1就表示a:，这样format %1就相当于format a:，于是上面的命令运行时实际执行的是format a:		
例2：C:根目录下一批处理文件名为t.bat，内容为:@echo offtype %1type %2那么运行C:\>t a.txt b.txt%1 : 表示a.txt%2 : 表示b.txt于是上面的命令将顺序地显示a.txt和b.txt文件的内容。		
		
		
将必要的参数（脚本中标红的部分）通过java命令传入启动程序，不同的参数值启动了不同的murex环境的客户端程序，参数值配置规则存放在渣打的文件服务器中。		
		
Take mxg_frd environment as example		
		login file server of SCB: magapp15a.uk
		magapp15a:mxghome
		magapp15a:cd /shared/opt/pos/mxg_fdr/live
		magapp15a:/shared/opt/pos/mxg_fdr/live$ less mxg2000_settings.sh (找到参数对应的值)
		###################################
		#!/bin/sh
		
		# Murex: Jun  2003
		# launchmxj.app 2.10 version
		# Mx G2000 Environment variables setup
		# set -x
		
		TODAY="`date +'%Y%m%d_%H%M%S'`"
		
		OS_TYPE=`uname`
		OS_PLATFORM=`uname -p`
		
		if [ "$OS_TYPE" = "SunOS" ]; then
		        case ${OS_PLATFORM} in
		        sparc)
		                JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		                ;;
		        i386)
		                JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_x86
		                ;;
		        esac
		fi
		if [ "$OS_TYPE" = "AIX" ]; then
		        JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		fi
		if [ "$OS_TYPE" = "HP-UX" ]; then
		        JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		fi
		if [ "$OS_TYPE" = "Linux" ]; then
		        JAVAHOME=/shared/opt/jdk/pos/j2sdk1.4.2_13_64/jre
		fi
		
		# Sybase home used to locate interface files and Open Client dynamic libraries
		case ${OS_PLATFORM} in
		sparc)
		        SYBASE=/shared/opt/sybase/openclient_pos/12.5
		        ;;
		i386)
		        SYBASE=/shared/opt/sybase/openclient_pos/12.5.x86
		        ;;
		esac
		
		export SYBASE
		LD_LIBRARY_PATH=$SYBASE/OCS-12_5/lib:/usr/openwin/lib:/usr/ccs/lib:/shared/opt/S
		CB/mx_api/live/lib
		export LD_LIBRARY_PATH
		
		if [ -d "/usr/sfw/lib" ]
		then
		        if [ `echo ${LD_LIBRARY_PATH}|grep "/usr/sfw/lib"|wc -l` = 0 ]
		        then
		                LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/sfw/lib
		        fi
		fi
		
		case ${OS_PLATFORM} in
		i386)
		        LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/shared/opt/SCB/SolarisStudioX86/live
		/lib
		        ;;
		esac
		
		export LD_LIBRARY_PATH
		
		# Oracle Home used to locate Oracle Client dynamic libraries
		ORACLE_HOME=
		#ORACLE_HOME=/local/oracle/app/oracle/product/9i
		export ORACLE_HOME
		NLS_LANG=AMERICAN_AMERICA.AL32UTF8
		export NLS_LANG
		
		#MXG2000_HOME=/apps/murfx2000_new
		#export MXG2000_HOME
		
		#MXJ_JDK_OR_JRE=jdk
		#MXJ_JDK_OR_JRE=jre
		
		# Define the encrypted password for the monitor
		MXJ_PASSWORD=001000f00010003000a000d000c0
		
		# Define your default Mx G2000 File Server environment
		# Warning : Take care of the MXJ_FILESERVER_HOST in case of running on different
		 host.
		MXJ_FILESERVER_HOST=magapp15a
		MXJ_FILESERVER_PORT=10251
		
		MXJ_FILESERVER_TIME_ADJUSTMENT=1
		# Optional arguments passed to the FileServer.
		# " " are mandatory if several args.
		FILESERVER_ARGS=
		
		# Define your default Mx G2000 XmlServer environment
		#Backward compatibility with Version 2.2.9
		#Leave it blank with 2.2.10.
		MXJ_HOST=
		MXJ_PORT=
		
		# Optional arguments passed to the XmlServer such as forcing attachment to
		# a specific IP adrress. " " are mandatory if several args.
		
		XML_SERVER_ARGS="-d64 -verbose:gc -Xloggc:logs/xmls.gc.log.${TODAY} -XX:+PrintGC
		Details -XX:+PrintGCTimeStamps -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi
		.dgc.server.gcInterval=3600000"
		
		# Define your default Mx G2000 Site and Hub environment
		#Define your site name, must be defined in site.mxres.
		#Leave it by default.
		MXJ_SITE_NAME=mxg_bodev2
		
		#Define your hub name, must be defined in site.mxres.
		#Leave it by default.
		MXJ_HUB_NAME=hub_gdc
		
		# Optional arguments passed to the Hub home such as forcing attachment to
		# a specific IP adrress. " " are mandatory if several args.
		#HUB_HOME_ARGS="-Djava.rmi.server.hostname=X.X.X.X"
		HUB_HOME_ARGS="-Xms512m -Xmx1024m"
		
		# Define your default Mx G2000 MxMlExchange environment
		# Optional arguments passed to the MxMlExchange Server.
		# " " are mandatory if several args.
		# --- Added on 25 Sep 2008 to capture MXML thread process
		## MXML_SERVER_ARGS="-Xloggc:mxml_gc_log.txt -XX:+PrintGCDetails"
		
		## CR: CMKL00000183191 :
		#MXML_SERVER_ARGS="-Xloggc:mxml.gc.log.${TODAY}.$$ -XX:+PrintGCDetails"
		MXML_SERVER_ARGS=
		
		# Define your default Mx G2000 Launcher environment
		# Optional arguments passed to launcher.
		# " " are mandatory if several args.
		LAUNCHER_ARGS=
		
		# Define your default Mx G2000 Murexnet environment
		# Warning : Must be the same as specified into the murexnet.mxres configuration
		file
		# by /IPHOST:namedhost:8000
		# The Murexnet usually run on 8000 port, but you can use another one.
		MUREXNET_PORT=21203
		
		#Optional arguments passed to the murexnet such as forcing attachment to
		# a specific IP adrress or logs." " are mandatory if several args.
		MUREXNET_ARGS="/IPALTADDR:magapp15a /IPLOG"
		
		#Rticachesession Display
		RTICACHESESSION_XWIN_DISP=`echo $DISPLAY | cut -d: -f1`
		
		EXTRA_ARGS="/MXJ_PING_TIME:60000 /MXJ_PING_CHECK:600000 /MXJ_MX_PING_TIME:60000
		/MXJ_MX_PING_CHECK:600000 /MXJ_LAUNCHER_PING_TIME:60000 /MXJ_LAUNCHER_PING_CHECK
		:600000"
		##########################
		
		
%TEMP%和%MXJ_BOOT%等变量是在哪里设置的？		

## Control-M 
ControlM(ctm)在file server上的根目录：/shared/opt/SCB/pos_eod/live, cd 到用户根目录下后，ctm命令可进入该目录。								
修改JOB Draft，即JOB的定义文件，然后将其导入到ControlM中。	主要修改以下6处							
	1.   APPLICATION="1512113"  ， 改成自己的bankID							
	2.   JOBNAME="DMART_RUBICON_TRADE" ,  指向CF文件的名称 ，但JOBNAME的值要大写，							
	3.    NODEID="magapp15a"  ,  指向UNIX Server的name，　为controlM系统提供链接哪个Unix服务器							
	4.    DATACENTER="MUREXUAT" ,  指定我们的ControlM JOBS 运行在哪个server上。这个server仅在controlＭ的group级别指定的，通常我们指定它为MUREXUAT							
	5.    CMDLINE="%%BASEPATH/scripts/eod_task.pl -jobcode=%%JOBNAME   -config=env802(com_daily/vareod,magapp15a:/shared/opt/SCB/pos_eod/POS_EOD_GL_4.1.0/config$ ls eod_main.*.cf)  -psserver X86(delete it?)"  ,  CMDLINE中指定JOB的运行程序脚本，该脚本将读取cf文件，job脚本可以运行在不同的Murex环境中，不同的Murex环境的环境变量是不同的，-config参数指定为环境变量文件的别名（别名为文件名的一部分）。不同的环境的环境变量文件定义在fileServer: /shared/opt/SCB/pos_eod/live/config/下。							
	6.    RUN_AS="posop" , 指定以哪个UNIX用户身份，即权限去执行该ＪＯＢ的CMDLINE.							
	7.     <JOB JOBNAME="PRO_TRADE_COM" …>  ，每个JOB都有一个JOBNAME标签，其值与该JOB的cf文件中定义的JOBNAME要一致，与cf文件名也一致，但cf文件名需要小写，而JOBNAME的值要大写。							
		cd命令进入用户根目录下后，输入jobdefs就可进入/shared/opt/SCB/pos_eod/live/config/jobdefs，ControlM  JOB的cf定义文件就在这个目录下，						
	8.    <RULE_BASED_CALENDARS NAME="BATCH_DAYS" /> 标签指明该JOB根据哪个Calenda Rule去执行。							
	9.   <JOB CONFCAL="BATCH_DAYS", ….>							
								
分组，　Entity folder								
								
								
								
								
								
Login ControlM system -> Planning -> create a new workspace -> import JOB draft file								
Unload all irrelevant job in the workspace -> click Check In -> click Order       Check In 和 Order的区别？？？？？？？？？？？？？？？？？？？？？								
Monitoring -> Recent ViewPoints -> All Jobs -> Application->Hirarchy/Application中输入查询的值（预定义的值是bankID)then open-> 								
	free all your jobs including the jobs' folder							
	Tools->QUANTITATIVE Resource -> delete the current resource and then add your own new resource ,source name equals with the  <QUANTITATIVE NAME="TONYPC$"…> defined in Job draft. Cpu choos 40 in day, 							
	find the most high level job -> right click waiting info-> Apply All -> right click on the job to  run now							
								
								
								
								
只有check　job 红了可以set OK,也可以根据check job的log去fix the check job								
								
								
								
								
								
								
								
								
								
								
								
control M cf file can configure in the following files.								
								
magapp15a:/shared/opt/SCB/pos_eod/live/config$ ls *dmart*dev1*								
								
								
								
								
对于pre-product env  （如frd, var等）的control M script file is under magapp15a:/shared/opt/SCB/pos_eod/live/config/jobdefs_preprod  not magapp15a:/shared/opt/SCB/pos_eod/live/config/jobdefs								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
ENV variable 								
$POS_MUREX_BASE  =>  /shared/opt/SCB/pos_murex/live								
								
								
								
								
cf file => DB configure file								
magapp15a:/shared/opt/SCB/poscommon/live/config$ more pos_db.cf								
								
magapp15a:/shared/home/posop$ $POSCOMMON_BASE/bin/posisql PROFILE_MXG_IRDFX_DEV9								
Msg 911, Level 11, State 2:								
Server 'mx9a_sql', Line 1:								
Attempt to locate entry in sysdatabases for database 'MXG_IRDFX_DEV9' by name								
failed - no entry found under that name. Make sure that name is entered								
properly.								
1>								
q								
Solution:								
1. cd /shared/opt/SCB/dbgen/live/bin								
2. rm jerry.sed								
3. vi jerry.dbo								
4. DBUpdateCF jerry.dbo jerry.cf à jerry.sed								
5. need copy 1								
								
								
								
7. cd /shared/opt/SCB/poscommon/live/config								
								
								
								
								
								
								
								
								
								
								
                1.  open your control M draft								
                2.  remove the key word FOLDER_ORDER_METHOD="SYSTEM"								
                3.  if need open a session to align our understanding, please let me know.								
								
								
								
								
$POS_VARUTILS_BASE								
/shared/opt/SCB/pos_varutils/live								
								
								
								
								
								
								
POST WEB 								
https://posweb.gdc.standardchartered.com:8002 								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
Tomcat								
/shared/opt/tomcat/spirit-tomcat/webapps/spirit								
								
Sybase								
/shared/opt/sybase/openclient_pos/12.0.0.3								
								
/opt/bin								
/shared/home/posop/scripts								
magapp15a:/opt$  cd $POSTOOLS								
								
								
								
								
								
								
								
Set a resource name(i.e. TONYZHU$) then distribute max resource to it(i.e. 80),then modify you control M draft  (QUANTITATIVE NAME="TONYZHU$")								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
								
Calendar: 								
Open Control-m  then navigate Planning ->  Tolls -> Calendars -> create new/ find out existing 'CalendarName'								
								
								
								
								
								
	Sno	DB Name	Profile Name	Login	Sybase server	Port	Physical Host	Virtual Host
	1	MXG	PROFILE_MXG	MUREXDB	mxg_sql	7000	mxgdb.gdc	db046a
	2	MXML	PROFILE_MXML	MUREXDB	mxg_sql	7000	mxgdb.gdc	db046a
	3	MXG_VAR	PROFILE_VAR	MUREXDB	pos_var_sql	8100	mxgvardb.gdc	db046b
	4	MXG_RPT	PROFILE_RPT	MUREXDB	mxg_rpt_sql	7300	mxgrptdb.gdc	db050a
	5	MXG_FDR	PROFILE_FDR	MUREXDB	mxg_fdr_sql	7200	mxgfdrdb.gdc	db050b
	6	INTDB	PROFILE_INTDB	posop	pos_sql	6100	posdb.gdc	db011b
	7	MXGDM	PROFILE_MXGDM	DMDBO	mxg_rpt_sql	7300	mxgrptdb.gdc	db050a
	8	MXGDM_RPT	PROFILE_MXGDM_RPT	DMDBO	mxg_rpt_sql	7200	mxgrptdb.gdc	db050a
	9	MXGDM_FDR	PROFILE_MXGDM_FDR	DMDBO	mxg_fdr_sql	7200	mxgfdrdb.gdc	db050b
								
								
								
								
								
magapp15a:/shared/opt/pos/mxg/mxg8_1/scb/reports/dates$ ls -rlt  date_td.txt								
-rwxr--r--   1 murex    murex         40 Jan 10  2016 date_td.txt								
								
								
								
								
								
								
								
								
Where to get the latest ControlM draft?								
/dev_vob/packages/poseidon/eod_auto/controlm/mxg2k_production.xml								
[1477277@dl1101 controlm]$ ls -rlt mxg2k_production.xml								
-r--r----- 1 1377478 gmdev 6856910 Oct 13 10:03 mxg2k_production.xml								
[1477277@dl1101 controlm]$ cp mxg2k_production.xml mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ ls -rlt mxg2k_production.xml.17Oct								
-r--r----- 1 1477277 1477277 6856910 Oct 17 07:56 mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ chmod 777 mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ ls -rlt mxg2k_production.xml.17Oct								
-rwxrwxrwx 1 1477277 1477277 6856910 Oct 17 07:56 mxg2k_production.xml.17Oct								
[1477277@dl1101 controlm]$ scp mxg2k_production.xml.17Oct posop@magapp15a.uk:/shared/home/posop/tonyzhu/mxg2k_production.xml.17Oct								
## Murex other
想知道JOB是否还在运行		
	执行SQL	
		sp_MxLock
	config login Murex -> publisher->Datamart->Job	
		
		
		
		
Joy's murex objects export from murex		
/shared/home/murex/su_roy_yu/project/rubicon/cm0000000679433/gl2.2.2/objects		
 
## 重启launcher
	MXG_FDR :-					
	Login: magapp15a: as murex					
	cd /shared/mrxdev_pos/mxg_fdr/live					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr stop					
	/shared/mrxdev_pos/mxg_pp_services_main.sh fdr start					


## /shared/mrxdev_pos/mxg_pp_services_main.sh
#!/bin/ksh

CURR_DIR=`pwd`
ENV=${1}
STATUS=${2}
MXML=${3:-"N"}
LAUNCHERS=${4:-"Y"}

case ${ENV} in 
mxg|var|fdr|rpt|purge)
	echo "valid environment provided ... continuing"
	;;
*)
	basename `dirname ${CURR_DIR} | sed -e "s;/live;;g"` | nawk -F "_" '{ print $NF }' | read -r ENV
	;;
esac

BINARY_PATH=${BINARY_PATH:-"/shared/mrxdev_pos"}

echo "-------------------------------------- [ ${ENV} ]"

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #
# function main_start()
# - - - - - - - - - - - - 
main_start() {
	./launchmxj.app -fs
	sleep 4
	case ${ENV} in
	mxg|purge)
		./launchmxj.app -xmlsnohub
		sleep 4
		./launchmxj.app -hub /MXJ_HUB_NAME:hub_gdc
		;;
	var|fdr|rpt)
		./launchmxj.app -xmls
		sleep 4
		;;
	esac

	./launchmxj.app -l
	./launchmxj.app -mxnet

	./launchmxj.app -s
}
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #
# function main_stop() {
# - - - - - - - - - - - - 
main_stop() {
	./launchmxj.app -mxnet -k
	./launchmxj.app -l -k
	case ${ENV} in 
	mxg)	
		./launchmxj.app -hub /MXJ_HUB_NAME:hub_gdc -k
		./launchmxj.app -xmlsnohub -k
		;;
	var|fdr|rpt)
		./launchmxj.app -xmls -k
		;;
	esac

	./launchmxj.app -fs -k
	./launchmxj.app -killall
	for all_hosts in `(echo magapp14a magapp15a; cat /shared/opt/SCB/dev_launchers/config/X86_ALL_HOSTS_PP.cf)`
	do
		ssh -q murex@${all_hosts} "cd ${CURR_DIR};./launchmxj.app -killall"
	done

	./launchmxj.app -s
}
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #
case ${STATUS} in
start)
	## Start main mandatory services
	main_start
	case ${ENV} in
	mxg)
		## Start MXML services for mxg env
		${BINARY_PATH}/mxg_pp_services_mxml.sh mxg start
		;;
	var|fdr|rpt|purge)
		echo "MXML services are not required for VAR/FDR/RPT environment .. continuing"
		;;
	esac
	if [ ${LAUNCHERS} == "Y" ]
	then
		## Start Launcher services for GUI and Batch
		${BINARY_PATH}/main_launchers.sh ${ENV} start

		case ${ENV} in
		fdr|rpt)
			## Start DealScanner Launcher services for FDR / RPT
			${BINARY_PATH}/dscan_launchers.sh ${ENV} start
			;;
		esac
	fi
;;
stop)
	case ${ENV} in
	fdr|rpt)
		## Stop DealScanner Launcher services for FDR / RPT
		${BINARY_PATH}/dscan_launchers.sh ${ENV} stop
		;;
	esac
	## Stop Launcher services for GUI and Batch
	${BINARY_PATH}/main_launchers.sh ${ENV} stop

	case ${ENV} in
	mxg)
		## Stop MXML services for mxg env
		${BINARY_PATH}/mxg_pp_services_mxml.sh mxg stop
		;;
	var|fdr|rpt|purge)
		echo "MXML services are not running for VAR/FDR/RPT environment .. continuing"
		;;
	esac

	## Stop main mandatory services
	main_stop

;;
test)
	./launchmxj.app -s
	;;
esac


## mxg_pp_services_mxml.sh
#!/bin/ksh

ENV=${1}
STATUS=${2}
CURR_DIR=`pwd`
BINARY_PATH=${BINARY_PATH:-"/shared/mrxdev_pos"}

case ${ENV} in
mxg|var|fdr|rpt|purge)
	echo "valid environment provided .. continuing.."
	;;
*)
	basename `dirname ${CURR_DIR} | sed -e "s;/live;;g"` | cut -d "_" -f2 | read -r ENV
	;;
esac

case ${STATUS} in
start)
	./launchmxj.app -fs
	sleep 4
	case ${ENV} in
	mxg)
		./launchmxj.app -mxml
		sleep 4
		
		./start_repository.sh
		sleep 4
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecmdex.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecri.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangedocdeal.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangefixings.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepayment.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepci.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf1.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf2.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangetds.mxres
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfinparser.mxres
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxdailyrepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfiniqrepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmarketdatarepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxssirepository.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxhurricanerepository.mxres
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcache.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcontribution.mxres
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxeventcapture.mxres
		sleep 4

		./mxg_start_soap_relay.sh
		sleep 4
		;;
	var|fdr|rpt)
		echo "MXML services are not required for VAR/FDR/RPT environment .. exiting"
		exit
		;;
	esac
	;;
stop)
	case ${ENV} in 
	mxg)	
		./mxg_stop_soap_relay.sh

		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxdailyrepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfiniqrepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmarketdatarepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxssirepository.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxhurricanerepository.mxres -k
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcache.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxcontribution.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxeventcapture.mxres -k
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecmdex.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangecri.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangedocdeal.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangefixings.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepayment.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangepci.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf1.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangestructconf2.mxres -k
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxmlexchangetds.mxres -k
		
		./launchmxj.app -l /MXJ_CONFIG_FILE:launchermxfinparser.mxres -k
		
		./stop_repository.sh

		./launchmxj.app -mxml -k
		./launchmxj.app -killall
		;;
	var|fdr|rpt)
		echo "MXML services are not required for VAR/FDR/RPT environment .. exiting"
		exit
		;;
	esac
;;
test)
	./launchmxj.app -s
	;;
esac

## main_launchers.sh
#!/bin/ksh
set -x

ENV=${1}
STATUS=${2}

CURR_DIR=`pwd`

BINARY_PATH=${BINARY_PATH:-"/shared/mrxdev_pos"}

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #

case ${ENV} in
mxg|var|fdr|rpt|purge)
	echo "valid environment provided .. continuing"
	;;
*)
	basename `dirname ${CURR_DIR} | sed -e "s;/live;;g"` | cut -d "_" -f2 | read -r ENV
	;;
esac
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - #

SPC_FILE="launcher_${ENV}_app0"
X86_FILE="launcher_${ENV}_ukspapmrx"

case ${STATUS} in 
start)
		case ${ENV} in
		mxg)
			for sname in `echo 56 69 76`
			do
				ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres" &
				ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres" &
			done
			;;
		esac
		for sname in `echo 59 66 67 68`
		do
			ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres" &
			ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres" &
		done
		
		for x86 in `echo 01 02 03 04 05 06 08 09 10 11`
		do
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}a.mxres" &
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}b.mxres" &
		done
		
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07a.mxres" &
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13a.mxres" &
		
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07b.mxres" &
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13b.mxres" &
		
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12a.mxres" &
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14a.mxres" & 
		
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12b.mxres" &
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14b.mxres" &

		for x86new in `echo 15 16 17 18`
		do
			A_FILE="${X86_FILE}${x86new}a.mxres"
			B_FILE="${X86_FILE}${x86new}b.mxres"
			d_x86new=$((x86new-3))
			d_host="ukspadmrx${d_x86new}a"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${A_FILE}" &
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${B_FILE}" &
		done
		for x86new in `echo 19a 19b 20a 20b`
		do
			case ${x86new} in
			19a)
				d_host=12a
				;;
			19b)
				d_host=13a
				;;
			20a)
				d_host=14a
				;;
			20b)
				d_host=15a
				;;
			esac
			P_FILE="${X86_FILE}${x86new}.mxres"
			d_host="ukspadmrx${d_x86new}"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${P_FILE}" &
		done
;;
stop)
		case ${ENV} in
		mxg)
			for sname in `echo 56 69 76`
			do
				ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres -k"
				ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres -k"
			done
			;;
		esac
		for sname in `echo 59 66 67 68`
		do
			ssh -q murex@magapp14a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}a.mxres -k"
			ssh -q murex@magapp15a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${SPC_FILE}${sname}b.mxres -k"
		done
		for x86 in `echo 01 02 03 04 05 06 08 09 10 11`
		do
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}a.mxres -k"
			ssh -q murex@ukspadmrx${x86}a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}${x86}b.mxres -k"
		done
		
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07a.mxres -k"
		ssh -q murex@ukspadmrx08a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13a.mxres -k"
		
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}07b.mxres -k"
		ssh -q murex@ukspadmrx09a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}13b.mxres -k"
		
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12a.mxres -k"
		ssh -q murex@ukspadmrx10a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14a.mxres -k"
		
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}12b.mxres -k"
		ssh -q murex@ukspadmrx11a "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${X86_FILE}14b.mxres -k"

		for x86new in `echo 15 16 17 18`
		do
			A_FILE="${X86_FILE}${x86new}a.mxres"
			B_FILE="${X86_FILE}${x86new}b.mxres"
			d_x86new=$((x86new-3))
			d_host="ukspadmrx${d_x86new}a"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${A_FILE} -k"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${B_FILE} -k"
		done
		for x86new in `echo 19a 19b 20a 20b`
		do
			case ${x86new} in
			19a)
				d_host=12a
				;;
			19b)
				d_host=13a
				;;
			20a)
				d_host=14a
				;;
			20b)
				d_host=15a
				;;
			esac
			P_FILE="${X86_FILE}${x86new}.mxres"
			d_host="ukspadmrx${d_x86new}"
			ssh -q murex@${d_host} "cd ${CURR_DIR};./launchmxj.app -l /MXJ_CONFIG_FILE:${P_FILE} -k"
		done
;;
esac


## magapp15a:/shared/opt/pos/mxg/mxg_bodev1$ less ./launchmxj.app
#!/bin/sh

# Murex: 13 Feb 2004
MAJOR_VERSION=2000.2.11
MINOR_VERSION=2.11
# Mx G2000 3.1.prod Startup/Stop script for servers clients and utils.
# set -x 0

#--- Operating System ---
OS_TYPE=`uname`
PLATFORM_TYPE=`uname -p`

if [ "$OS_TYPE" = "AIX" ]; then
  MACHINE_NAME=`hostname -s`
else
  MACHINE_NAME=`hostname`
fi

echo "$1"
if [ "${MACHINE_NAME}" = "gmsitnfs" ]
then
	if [ "$1" != "-s" ]
	then
		exit
	fi
fi

##################################################

##################################################
_LS=/usr/bin/ls
_LS_L="/usr/bin/ls -l"
_PS=/usr/bin/ps

case $PLATFORM_TYPE in
sparc)
    #_TEE="/shared/opt/pos/mxg/live/mxtee -t"
    _TEE="/shared/opt/SCB/dev_launchers/scripts/mxtee -t"
    ;;
*)
    _TEE="/usr/bin/tee"
    ;;
esac

#_TEE="/shared/opt/pos/mxg/live/mxtee -t"
#_TEE=/usr/bin/tee
_AWK=/usr/bin/awk
_ECHO=echo

if [ "$OS_TYPE" = "Linux" ]; then
_LS=/bin/ls
_LS_L="/bin/ls -l"
_PS=/bin/ps
_AWK=/bin/awk
_ECHO="echo -e"
fi

_ID=`id`
if [ "$OS_TYPE" = "AIX" ]; then
  USER_NAME=`echo $_ID| sed s/\(/" "/| sed s/\)/" "/| $_AWK '{printf "%s", substr($2,1,8)}'`
else
  USER_NAME=`echo $_ID| sed s/\(/" "/| sed s/\)/" "/| $_AWK '{print $2}'`
fi

PATH=.:$PATH
export PATH

##########################################
# Common application argument environnemnt 
# Set it up here: Eventually modify the following lines according to your needs.
##########################################

# Java Debug Log Level
# 0|1|2|3|4
MXJ_LOG_LEVEL=2

# Path to store logs and PID files
LOG_PATH=logs

#Append log file at start/stop command
APPEND_LOG=1
#Create new empty log file at start/stop command
#APPEND_LOG=0

#Time loop value for the status option
#in seconds
LOOP_TIME=60

#Set File Desc default value
FD_LIMIT=2048
#FD_LIMIT=1024

# Settings for the Mx G2000 File Server.
########################################
FILESERVER_PATH=fs
MXJ_HTTP_JAR=mxjhttp.jar
# This file contains the complete path to the .jar files that are provided by the Mx G2000 File server.
# It is also used as a parameter for the other servers and the client.
MXJ_JAR_FILE=murex.download.service.download
MXJ_GUI_JAR_FILE=murex.download.guiclient.download
MXJ_FILESERVER_CONFIG_FILE=fileserver.xml

# Settings for the Mx G2000 Xml Server.
########################################
# In case of no optional flags from the setting file
# set the var to null. Do not modify here.
XML_SERVER_ARGS=

# The xmlserver.mxres allow you to specify ports to use for the xmlserver.
#MXJ_XMLSERVER_CONFIG_FILE=public.mxres.xmlserver.xmlserver.mxres

# XmlServer Stat
#MXJ_STAT_FILE_NAME=stat
#MXJ_STAT_PERIOD=10000

# Settings for the MXML Server.
########################################
# The following setting is overrwitten by the one defined in mxg2000_settings, if exists.
DEFAULT_MXML_SERVER_ARGS=""
## DEFAULT_MXML_JVM_ARGS="-Xms256M -Xmx512M -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000"
# CR: CMKL00000183191
DEFAULT_MXML_JVM_ARGS="-d64 -Xmx2048M -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000"
if [ "$OS_TYPE" = "SunOS" ]; then
   DEFAULT_MXML_JVM_ARGS="-server $DEFAULT_MXML_JVM_ARGS"
fi
MXML_SERVER_ARGS="$DEFAULT_MXML_SERVER_ARGS"
MXML_JVM_ARGS="$DEFAULT_MXML_JVM_ARGS"

# Settings for the MDCS Server.
########################################
# The following setting is overrwitten by the one defined in mxg2000_settings, if exists.
DEFAULT_MDCS_SERVER_ARGS=""
DEFAULT_MDCS_JVM_ARGS="-Xmx2g -Dsun.rmi.dgc.client.gcInterval=36000000 -Dsun.rmi.dgc.server.gcInterval=36000000"
if [ "$OS_TYPE" = "SunOS" ]; then
   DEFAULT_MDCS_JVM_ARGS="-server $DEFAULT_MDCS_JVM_ARGS"
fi
MDCS_SERVER_ARGS="$DEFAULT_MDCS_SERVER_ARGS"
MDCS_JVM_ARGS="$DEFAULT_MDCS_JVM_ARGS"

# Settings for the MDRS Server.
########################################
# The following setting is overrwitten by the one defined in mxg2000_settings, if exists.
DEFAULT_MDRS_SERVER_ARGS=""
DEFAULT_MDRS_JVM_ARGS="-Xmx512M -Dsun.rmi.dgc.client.gcInterval=36000000 -Dsun.rmi.dgc.server.gcInterval=36000000"
MDRS_SERVER_ARGS="$DEFAULT_MDRS_SERVER_ARGS"
MDRS_JVM_ARGS="$DEFAULT_MDRS_JVM_ARGS"

# Settings for all applications.
################################
# Default setting file.
SETTINGS_FILE=mxg2000_settings.sh

# Warning the MXJ_BOOT_JAR file MUST be on the same directory as the executable.
MXJ_BOOT_JAR=mxjboot.jar

# Define your default Launcher environement
# The launcherall.mxres descibe the flags you use to launch the application itself.
MXJ_CONFIG_FILE=public.mxres.common.launcherall.mxres

# Define your site name
# The site.mxres descibe the site itself.
MXJ_SITE_NAME=site1

# Define your hub name
# The site.mxres descibe the site itself.
MXJ_HUB_NAME=hub1

# Define your default MxMlExchange environement
# The file launchermxmlexchangeall.mxres describe the flags you use to launch the application itself.
MXJ_MXMLEX_CONFIG_FILE=public.mxres.common.launchermxmlexchangeall.mxres
MXJ_MXMLEX_CONFIG_FILE_SECONDARY=public.mxres.common.launchermxmlexchangesecondary.mxres
MXJ_MXMLEX_CONFIG_FILE_SPACES=public.mxres.common.launchermxmlexchangespaces.mxres

# Define your default Mx Contribution environement
MXJ_CONTRIBUTION_CONFIG_FILE=public.mxres.common.launchermxcontribution.mxres

# Define your default MDCS environement
# The file launcherwarehouse.mxres describe the flags you use to launch the application itself.
MXJ_MDCS_CONFIG_FILE=public.mxres.common.launchermxcache.mxres

# Define your default MDRS environement
# The file launchermxmarketdatarepository.mxres describe the flags you use to launch the application itself.
MXJ_MDRS_CONFIG_FILE=public.mxres.common.launchermxmarketdatarepository.mxres

#Define your default ActivityFeeder environment
# The file launchermxactivityfeeders.mxres describe the flags you use to launch the application itself.
MXJ_ACTIVITY_FEEDER_CONFIG_FILE=public.mxres.common.launchermxactivityfeeders.mxres

# Setting for murexnet.
#######################
#Port used by the murexnet, defined here if not setted in mxg2000_settings file
#(backward compatibility)
MUREXNET_PORT=8000

# In case of no optional flags from the setting file
# set the var to null. Do not modify here.
MUREXNET_ARGS=

# Settings for clients.
#######################
# Define your default client  environement
MXJ_PLATFORM_NAME=MX
MXJ_PROCESS_NICK_NAME=MX
# Define your default Client macro XML file
MXJ_SCRIPT=key.xml

###################################
# End of user definables settings # 
###################################

######################################################################
# Sourcing Setting File and Setting env.
######################################################################
Setting_Env() {  
# Java and DATABASE SERVER settings file sourced.
# Can be overwritten by option -i:setting_file

if [ ! -f `dirname $0`/$SETTINGS_FILE ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Environnement settings file: $SETTINGS_FILE not found !"
   $_ECHO "          Or : forget, -i:setting_file."
   exit 1
fi
 
. `dirname $0`/$SETTINGS_FILE
RTISESSION_XWIN_DISP=$DISPLAY
echo $RTISESSION_XWIN_DISP
RTICACHESESSION_XWIN_DISP=$DISPLAY
echo $RTICACHESESSION_XWIN_DISP

#--- OS LIBARY environment.
###########################
# Path to OS Library.
if [ "$OS_TYPE" = "SunOS" ]; then
   LD_LIBRARY_PATH=/usr/lib/lwp:$LD_LIBRARY_PATH
   export LD_LIBRARY_PATH
fi

#--- Java environment.
######################
# Path to the Java binaries, the 'libjvm.so' dynamic library
# Set it up here: Uncomment and/or modify the following lines according to your setup.

if [ "$JAVAHOME" = "" ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Please specify the Java environment "
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ "$OS_TYPE" = "SunOS" ]; then
   if [ -d $JAVAHOME/jre ]; then
   case $PLATFORM_TYPE in
        sparc )
        JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/sparc
        ;;
        i386 )
        JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/i386
        ;;
   esac
   else
   case $PLATFORM_TYPE in
        sparc )
        JAVA_LIBRARY_PATH=$JAVAHOME/lib/sparc
        ;;
        i386 )
        JAVA_LIBRARY_PATH=$JAVAHOME/lib/i386
        ;;
   esac
   fi
   PATH=$JAVAHOME/bin:$PATH
   LD_LIBRARY_PATH=$JAVA_LIBRARY_PATH:$LD_LIBRARY_PATH
   LIB_EXT=so
   export LD_LIBRARY_PATH
   # Set Numeric format for SUN
   LC_NUMERIC=en_US
   export LC_NUMERIC
fi
if [ "$OS_TYPE" = "AIX" ]; then
   JAVA_LIBRARY_PATH=$JAVAHOME/jre/bin/classic
   PATH=$JAVAHOME/jre/sh:$JAVAHOME/sh:$PATH
   LIBPATH=$JAVA_LIBRARY_PATH:$JAVAHOME/jre/bin:$LIBPATH
   LIB_EXT=a
   export LIBPATH
   # Set Numeric format for IBM
   LANG=en_US
   export LANG
   export AIXTHREAD_SCOPE=S
   export AIXTHREAD_MUTEX_DEBUG=OFF
   export AIXTHERAD_RWLOCK_DEBUG=OFF
   export AIXTHREAD_COND_DEBUG=OFF
   if [ "$LDR_CNTRL" != "" ] ; then
      LDR_CNTRL_MAXDATA=`echo $LDR_CNTRL | grep "MAXDATA"`
	  $_ECHO "WARNING: LDR_CNTRL is set with MAXDATA value:"$LDR_CNTRL
   fi
   # Settings to force JAVA to go through ipv4 stack instead of ipv6.
   export IBM_JAVA_OPTIONS="-Djava.net.preferIPv4Stack=true  ${IBM_JAVA_OPTIONS}"  
fi
if [ "$OS_TYPE" = "HP-UX" ]; then
   JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/PA_RISC2.0/hotspot
   PATH=$JAVAHOME/bin:$PATH
   SHLIB_PATH=$JAVA_LIBRARY_PATH:$SHLIB_PATH
   LIB_EXT=sl
   export SHLIB_PATH
fi
if [ "$OS_TYPE" = "Linux" ]; then
   JAVA_VENDOR=`$JAVAHOME/jre/bin/java -version 2>&1 | grep IBM` 
   if [ $? -ne 0 ] ; then 
      #Assume we are using SUN JVM
      JAVA_LIBRARY_PATH=$JAVAHOME/jre/lib/i386/client
   else
      #Assume we are using IBM JVM
      JAVA_LIBRARY_PATH=$JAVAHOME/jre/bin/client
   fi
   LD_LIBRARY_PATH=$JAVA_LIBRARY_PATH:$LD_LIBRARY_PATH
   PATH=$JAVAHOME/bin:$PATH
   LIB_EXT=so
   LC_NUMERIC=en_US
   export LIB_EXT LC_NUMERIC LD_LIBRARY_PATH
fi

export PATH

#--- Sybase environment used only by the launcher.
##################################################

if [ "$SYBASE" != "" ] ; then
case $OS_TYPE in
        SunOS )
        SYBASE_OCS=OCS-12_5
	LD_LIBRARY_PATH=$SYBASE/$SYBASE_OCS/lib:$SYBASE/lib:.:$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
        ;;
        AIX )
        SYBASE_OCS=OCS-12_5
	LIBPATH=$SYBASE/$SYBASE_OCS/lib:.:$LIBPATH
        export LIBPATH
        ;;
        HP-UX )
        SYBASE_OCS=OCS-12_5
	SHLIB_PATH=$SYBASE/$SYBASE_OCS/lib:$SYBASE/lib:.:$SHLIB_PATH
        export SHLIB_PATH
        ;;
        Linux )
	SYBASE_OCS=OCS-12_5
	LD_LIBRARY_PATH=$SYBASE/$SYBASE_OCS/lib:$SYBASE/lib:.:$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
        ;;
        * )
        $_ECHO "Warning : Do not know how to handle this OS type $OS_TYPE."
        ;;
esac
export SYBASE_OCS
fi

#--- Oracle environment.
##################################################

if [ "$ORACLE_HOME" != "" ] ; then
case $OS_TYPE in  
	SunOS )
	LD_LIBRARY_PATH=$ORACLE_HOME/lib32:/usr/ccs/lib:.:$LD_LIBRARY_PATH
	export LD_LIBRARY_PATH
	LD_LIBRARY_PATH_64=$ORACLE_HOME/lib:$LD_LIBRARY_PATH_64
	export LD_LIBRARY_PATH_64
	PATH=$ORACLE_HOME/bin:$PATH
	export PATH
    	;;
	AIX )
	LIBPATH=$ORACLE_HOME/lib32:$ORACLE_HOME/lib:.:$LIBPATH
	export LIBPATH
        ;;
	HP-UX )
        SHLIB_PATH=$ORACLE_HOME/lib32:$ORACLE_HOME/lib:.:$SHLIB_PATH
        export SHLIB_PATH
        ;;
	Linux )
        LD_LIBRARY_PATH=$ORACLE_HOME/lib32:$ORACLE_HOME/lib:/usr/ccs/lib:.:$LD_LIBRARY_PATH
        export LD_LIBRARY_PATH
        ;;
        * )
        $_ECHO "Warning : Do not know how to handle this OS type $OS_TYPE."
        ;;
esac
fi

#--- File descriptors configuration.
################################################
ulimit -n $FD_LIMIT

FILE_DESC=`ulimit -n`

if [ $FILE_DESC -lt $FD_LIMIT ]; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Can not set file descriptors to $FD_LIMIT for current shell."
   exit 1
fi
} # End of Setting_Env

#--- Copying latest mxjboot.jar if needed.
################################################

if [ -f `dirname $0`/jar/$MXJ_BOOT_JAR ] ; then
   diff ./$MXJ_BOOT_JAR jar/$MXJ_BOOT_JAR >/dev/null
   if [ $? -ne 0 ] ; then
      cp jar/$MXJ_BOOT_JAR .
   fi
fi

# End of configuration, nothing to modify below.

######################################################################
# Mx G2000 File server
######################################################################
Fileserver() {

Define_Log_File_Name fileserver
Init_Log_File

if [ "$FILESERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$FILESERVER_ARGS"
fi

cd $FILESERVER_PATH

JAVA_CMD="java $JVM_OPTION $FILESERVER_ARGS -cp $MXJ_HTTP_JAR murex.http.fileserver.FileServerJar /MXJ_PORT:$MXJ_FILESERVER_PORT /MXJ_JAR_FILE:$MXJ_JAR_FILE /MXJ_CONFIG_FILE:$MXJ_FILESERVER_CONFIG_FILE $EXTRA_ARGS"

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>../$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a ../$LOG_PATH/$LOG_FILE.log &
cd ..
Update_Log_Pid_Files $! 0

}

######################################################################
# MXJXmlserver
######################################################################
Xmlserver() {


Define_Log_File_Name xmlserver
Init_Log_File 

if [ "$XML_SERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$XML_SERVER_ARGS"
fi
if [ "$OS_TYPE" = "SunOS" ]; then
   JVM_OPTION=" -server $JVM_OPTION"
fi

JAVA_CMD="java $JVM_OPTION $XML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.home.XmlHomeStartAll /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_HUB_NAME:$MXJ_HUB_NAME.$MXJ_SITE_NAME  $EXTRA_ARGS "

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# HubHome 
######################################################################
HubHome() {
#$_ECHO "hub home:/MXJ_HUB_NAME:$MXJ_HUB_NAME.$MXJ_SITE_NAME"
Define_Log_File_Name hubhome 
Init_Log_File

if [ "$XML_SERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$HUB_HOME_ARGS"
fi
 
JAVA_CMD="java $JVM_OPTION $HUB_HOME_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.hub.HubHome /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_HUB_NAME:$MXJ_HUB_NAME.$MXJ_SITE_NAME $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MXJXMLserver  XMLSNOHUB
######################################################################
XmlserverNoHub(){

Define_Log_File_Name xmlservernohub
Init_Log_File 

if [ "$XML_SERVER_ARGS" != "" ];then
   $_ECHO "Using specific args:$XML_SERVER_ARGS"
fi
if [ "$OS_TYPE" = "SunOS" ]; then
   JVM_OPTION=" -server $JVM_OPTION"
fi

JAVA_CMD="java  $JVM_OPTION $XML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.home.XmlHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS "

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MXJTransactionManager
######################################################################
TransactionManager() {


Define_Log_File_Name transactionmanager
Init_Log_File 

JAVA_CMD="java -server $JVM_OPTION $XML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.tm.TransactionManagerHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS"

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}


######################################################################
# Launcher 
######################################################################
Launcher(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !" 
   exit 1
fi 

   JAVA_CLASSIC=

#ldd mx

Define_Log_File_Name launcher
Init_Log_File 

if [ "$LAUNCHER_ARGS" != "" ];then
   $_ECHO "Using specific args:$LAUNCHER_ARGS"
fi

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar:./jar/xml-apis.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl "

JAVA_CMD="java $JAVA_CLASSIC $JVM_OPTION $LAUNCHER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MxMlExchange  
######################################################################
Mxmlexchange(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MxMlExchange Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=


if [ "$MXML_SERVER_ARGS" != "$DEFAULT_MXML_SERVER_ARGS" ];then
   $_ECHO "Using specific MXML_SERVER_ARGS args: $MXML_SERVER_ARGS"
else
   $_ECHO "Using default MXML_SERVER_ARGS args: $MXML_SERVER_ARGS"
fi
 
if [ "$MXML_JVM_ARGS" != "$DEFAULT_MXML_JVM_ARGS" ];then
   $_ECHO "Using specific MXML_JVM_ARGS args: $MXML_JVM_ARGS"
else
   $_ECHO "Using default MXML_JVM_ARGS args: $MXML_JVM_ARGS"
fi

Define_Log_File_Name mxmlexchange
Init_Log_File

JAVA_CMD="java -verbose:gc -Xloggc:${LOG_PATH}/${LOG_FILE}.gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps $MXML_JVM_ARGS -Xbootclasspath/p:jar/xerces.jar:jar/xml-apis.jar:jar/xalan.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl $JAVA_CLASSIC $MXML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

Define_Log_File_Name mxmlexchangesecondary
Init_Log_File

JAVA_CMD="java -verbose:gc -Xloggc:${LOG_PATH}/${LOG_FILE}.gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps $MXML_JVM_ARGS -Xbootclasspath/p:jar/xerces.jar:jar/xml-apis.jar:jar/xalan.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl $JAVA_CLASSIC $MXML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE_SECONDARY $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

Define_Log_File_Name mxmlexchangespaces
Init_Log_File

JAVA_CMD="java -verbose:gc -Xloggc:${LOG_PATH}/${LOG_FILE}.gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps $MXML_JVM_ARGS -Xbootclasspath/p:jar/xerces.jar:jar/xml-apis.jar:jar/xalan.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl $JAVA_CLASSIC $MXML_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE_SPACES $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MDCS
######################################################################
MDCS_CACHE(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MDCS Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=

Define_Log_File_Name mdcs
Init_Log_File

if [ "$MDCS_SERVER_ARGS" != "$DEFAULT_MDCS_SERVER_ARGS" ];then
   $_ECHO "Using specific MDCS_SERVER_ARGS args: $MDCS_SERVER_ARGS"
else
   $_ECHO "Using default MDCS_SERVER_ARGS args: $MDCS_SERVER_ARGS"
fi

if [ "$MDCS_JVM_ARGS" != "$DEFAULT_MDCS_JVM_ARGS" ];then
   $_ECHO "Using specific MDCS_JVM_ARGS args: $MDCS_JVM_ARGS"
else
   $_ECHO "Using default MDCS_JVM_ARGS args: $MDCS_JVM_ARGS"
fi

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar "

JAVA_CMD="java $MDCS_JVM_ARGS $JVM_OPTION $JAVA_CLASSIC $MDCS_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MDCS_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a /$LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MDRS
######################################################################
MDRS_ENGINE(){

if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MDRS Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

JAVA_CLASSIC=

Define_Log_File_Name mdrs
Init_Log_File

if [ "$MDRS_SERVER_ARGS" != "$DEFAULT_MDRS_SERVER_ARGS" ];then
   $_ECHO "Using specific MDRS_SERVER_ARGS args: $MDRS_SERVER_ARGS"
else
   $_ECHO "Using default MDRS_SERVER_ARGS args: $MDRS_SERVER_ARGS"

fi
if [ "$MDRS_JVM_ARGS" != "$DEFAULT_MDRS_JVM_ARGS" ];then
   $_ECHO "Using specific MDRS_JVM_ARGS args: $MDRS_JVM_ARGS"
else
   $_ECHO "Using default MDRS_JVM_ARGS args: $MDRS_JVM_ARGS"
fi

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar "

JAVA_CMD="java $MDRS_JVM_ARGS $JVM_OPTION $JAVA_CLASSIC $MDRS_SERVER_ARGS -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_MDRS_CONFIG_FILE $EXTRA_ARGS"

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# Olk
######################################################################
Olk(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: Olk Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

OLK_EXEC_FILE=olk_exec.sh

if [ ! -f `dirname $0`/$OLK_EXEC_FILE ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Executable olk command file: $OLK_EXEC_FILE not found !"
   exit 1
fi
if [ ! -x `dirname $0`/$OLK_EXEC_FILE ] ; then
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          Olk command file: $OLK_EXEC_FILE not executable, change rights !"
   exit 1
fi

Define_Log_File_Name olk
Init_Log_File 
. `dirname $0`/$OLK_EXEC_FILE $EXTRA_ARGS \
2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 1

}

######################################################################
# RtImport
######################################################################
RtImport(){
#Params : session or start or stop or rtifxg
MXJ_RTIMPORT_CONFIG_PATH=public.mxres.mxcontribution.

case "$1" in
'session')

#Used to set the display value
$_ECHO $RTISESSION_XWIN_DISP >./RTISESSION_XWIN_DISP.tmp
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportsession.mxres $EXTRA_ARGS

        ;;

'rticachesession')
#set -x
#Used to set the display value
$_ECHO $RTICACHESESSION_XWIN_DISP >./RTICACHESESSION_XWIN_DISP.tmp
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportcachesession.mxres $EXTRA_ARGS

 ;;

'rticachestart')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportcachestart.mxres $EXTRA_ARGS

        ;;

'rticachestop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportcachestop.mxres $EXTRA_ARGS

        ;;

'rtiexportresults')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportexportresults.mxres $EXTRA_ARGS

        ;;

'start')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportstart.mxres $EXTRA_ARGS

        ;;
'stop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportstop.mxres $EXTRA_ARGS
        
        ;;

'fxgsession')
#set -x
#Used to set the display value
$_ECHO $RTICACHESESSION_XWIN_DISP >./RTICACHESESSION_XWIN_DISP.tmp
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportfixingsession.mxres $EXTRA_ARGS

        ;;
'fxgstart')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportfixingstart.mxres $EXTRA_ARGS

        ;;
'fxgstop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_RTIMPORT_CONFIG_PATH"rtimportfixingstop.mxres $EXTRA_ARGS

        ;;
esac

}

######################################################################
# MxParam
######################################################################
MxParam(){
#Params : start or stop
MXJ_MXPARAM_CONFIG_PATH=public.mxres.mxcontribution.
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: Mxparam Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

case "$1" in
'start')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxparamstop.mxres $EXTRA_ARGS

        sleep 15

        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxpmanagementstop.mxres $EXTRA_ARGS

        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxpmanagementstart.mxres $EXTRA_ARGS


        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxparamstart.mxres $EXTRA_ARGS &

        ;;
'stop')
        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxparamstop.mxres $EXTRA_ARGS

        sleep 15

        java -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD \
/MXJ_CONFIG_FILE:"$MXJ_MXPARAM_CONFIG_PATH"mxpmanagementstop.mxres $EXTRA_ARGS
        
        ;;
esac

}

######################################################################
# Murexnet 
######################################################################
Murexnet() {

Define_Log_File_Name murexnet
Init_Log_File 

MXNET_MBX_CNT="mbx$MUREXNET_PORT.cnt"
MXNET_MBX_CHK="mbx$MUREXNET_PORT.chk"
MXNET_MBX_EVT="mbx$MUREXNET_PORT.evt"
MXNET_MBX_IND="mbx$MUREXNET_PORT.ind"
MXNET_MBX_NOD="mbx$MUREXNET_PORT.nod"
MXNET_MBX_ERR="mbx$MUREXNET_PORT.err"
MXNET_MBX_IERR="mbx$MUREXNET_PORT.ierr"

if [ -f $MXNET_MBX_CNT ] ; then
  rm $MXNET_MBX_CNT
fi
if [ -f $MXNET_MBX_CHK ] ; then
  rm $MXNET_MBX_CHK
fi
if [ -f $MXNET_MBX_EVT ] ; then
  rm $MXNET_MBX_EVT
fi
if [ -f $MXNET_MBX_IND ] ; then
  rm $MXNET_MBX_IND
fi
if [ -f $MXNET_MBX_NOD ] ; then
  rm $MXNET_MBX_NOD
fi
if [ -f $MXNET_MBX_ERR ] ; then
  rm $MXNET_MBX_ERR
fi
if [ -f $MXNET_MBX_IERR ] ; then
  rm $MXNET_MBX_IERR
fi
$_ECHO "Cleanup done."

if [ "$MUREXNET_ARGS" != "" ];then
   $_ECHO "Using specific args:$MUREXNET_ARGS"
fi

MXNET_CMD="./murexnet /ipaddr:$MUREXNET_PORT /stdout:stdout /stderr:stderr $MUREXNET_ARGS $EXTRA_ARGS"

$_ECHO "Murexnet cmd:\n$MXNET_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$MXNET_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MxContribution
######################################################################
MxContribution(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MxContribution Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=

if [ "$OS_TYPE" = "SunOS" ]; then
   JVM_OPTION=" -server $JVM_OPTION"
fi

#ldd mx

Define_Log_File_Name mxcontrib
Init_Log_File

JAVA_CMD="java $JVM_OPTION -Xmx150M -Xms150M $JAVA_CLASSIC -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_CONTRIBUTION_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a ../$LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# MxActivityFeeder
######################################################################
MxActivityFeeder(){
if [ "$SYBASE" = "" ] && [ "$ORACLE_HOME" = "" ] ; then
   $_ECHO "Mx G2000: MxActivityFeeder Launcher Fatal ERROR: please specify the SYBASE or ORACLE environment variable"
   $_ECHO "          in the $SETTINGS_FILE script file"
   exit 1
fi

if [ ! -f "$JAVA_LIBRARY_PATH/libjvm.$LIB_EXT" ] ; then
   $_ECHO "Mx G2000: Launcher Fatal ERROR: "
   $_ECHO "          $JAVA_LIBRARY_PATH/libjvm.$LIB_EXT not found "
   $_ECHO "          file libjvm.$LIB_EXT not found, please check your $SETTINGS_FILE file !"
   exit 1
fi

   JAVA_CLASSIC=

#ldd mx

Define_Log_File_Name feeder
Init_Log_File

JVM_OPTION=$JVM_OPTION" -Xbootclasspath/p:./jar/xerces.jar:./jar/xalan.jar:./jar/jaxp-api.jar:./jar/xml-apis.jar -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl "

JAVA_CMD="java $JVM_OPTION -Xmx150M -Xms150M $JAVA_CLASSIC -cp $MXJ_BOOT_JAR -Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader /MXJ_CLASS_NAME:murex.xml.server.launcher.LauncherHome /MXJ_SITE_NAME:$MXJ_SITE_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_CONFIG_FILE:$MXJ_ACTIVITY_FEEDER_CONFIG_FILE $EXTRA_ARGS" 

$_ECHO "Java cmd:\n$JAVA_CMD\n" >>$LOG_PATH/$LOG_FILE.log
$JAVA_CMD 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log &
Update_Log_Pid_Files $! 0

}

######################################################################
# Client
######################################################################
Client() {

MXJ_BOOT_JAR=$MXJ_BOOT_JAR
java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_GUI_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.gui.xml.XmlGuiClientBoot /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS
exit $?
}

######################################################################
# Client with macro
######################################################################
ClientMacro() {

MXJ_BOOT_JAR=$MXJ_BOOT_JAR
java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.gui.api.script.ApiScript /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME \
/MXJ_SCRIPT_READ_FROM:$MXJ_SCRIPT /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL $EXTRA_ARGS
exit $?
}


######################################################################
# Monitor
######################################################################
Monitor() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.MonitorDisplay /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD  $EXTRA_ARGS
exit $?
}
######################################################################
# Script Monitor
######################################################################
Script_Monitor() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.server.monitor.monitordisplay.Monitor /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL /MXJ_PASSWORD:$MXJ_PASSWORD /MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE $EXTRA_ARGS
exit $?
}
######################################################################
# Remote Diagnostic Tool
######################################################################
MxRdt() {

OutputDir=logs/MxRDT
export OutputDir
MxRDT_LogFile=${OutputDir}/MxRDT_log.html

if [ ! -d "$OutputDir" ]; then
        mkdir "$OutputDir"
        Result=$?
        if [ "$Result" != "0" ]; then
           echo "Error creating output directory. Exit."
           exit $Result
        fi
else
        touch $MxRDT_LogFile
        Result=$?
        if [ "$Result" != "0" ]; then
          echo "Could not write in output directory. Exit. "
          exit $Result
        fi
fi

echo '<H3> <FONT COLOR="#336699"> MxRDT Log </FONT> </H4> <pre>' > $MxRDT_LogFile
. ./mxrdt 2>&1 | tee -a $MxRDT_LogFile
Result=$?
# echo '</pre>' >> $MxRDT_LogFile
exit $Result
}
######################################################################
# XmlRequestScript
######################################################################
XmlRequestScript() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.client.xmllayer.script.XmlRequestScript /MXJ_SITE_NAME:$MXJ_SITE_NAME \
/MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL \
/MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE $EXTRA_ARGS
exit $?
}
######################################################################
# PasswordEncryption
######################################################################
PasswordEncryption() {

java $JVM_OPTION -cp $MXJ_BOOT_JAR \
-Djava.rmi.server.codebase=http://$MXJ_FILESERVER_HOST:$MXJ_FILESERVER_PORT/$MXJ_JAR_FILE murex.rmi.loader.RmiLoader \
/MXJ_CLASS_NAME:murex.xml.cryptography.GuiPassword  $EXTRA_ARGS
exit $?
}

######################################################################
#
# Beginning of script
#
######################################################################
error(){
        $_ECHO "$0: Error: $*"
}

Define_Log_File_Name() {
# Params : ID of Service 
ID=$1
LOG_FILE=

case $ID in  
        fileserver )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_FILESERVER_PORT
        ;;
        xmlserver )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_HUB_NAME.$MXJ_SITE_NAME
        ;;
        xmlservernohub )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME
        ;;
        hubhome )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_HUB_NAME.$MXJ_SITE_NAME
        ;; 
        transactionmanager )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_PORT.$MXJ_SITE_NAME
        ;;
        launcher )
                if [ "$MXJ_INSTALLATION_CODE" = "" ] ; then
                   LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_CONFIG_FILE
                else
                   LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_CONFIG_FILE.$MXJ_INSTALLATION_CODE
                fi
        ;;
        mxmlexchange )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MXMLEX_CONFIG_FILE
        ;;
        mxmlexchangesecondary )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MXMLEX_CONFIG_FILE_SECONDARY
        ;;
        mxmlexchangespaces )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MXMLEX_CONFIG_FILE_SPACES
        ;;       
        murexnet )
                LOG_FILE=$MACHINE_NAME.$ID.$MUREXNET_PORT
        ;;
        mdcs )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MDCS_CONFIG_FILE
        ;;
        mdrs )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_MDRS_CONFIG_FILE
        ;;
        olk )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        mxparam )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        rtimport )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        mxcontrib )
                LOG_FILE=$MACHINE_NAME.$ID
        ;;
        feeder )
                LOG_FILE=$MACHINE_NAME.$ID.$MXJ_SITE_NAME.$MXJ_ACTIVITY_FEEDER_CONFIG_FILE
         ;;
        * )
                $_ECHO "Warning : Do not know how to handle this service."
                $_ECHO "          No way to stop it except manually."
        ;;
esac

}

Init_Log_File() {
#Params : None

if [ ! -d $LOG_PATH ] ; then
   mkdir $LOG_PATH
fi
if [ ! -d $LOG_PATH ] ; then
   $_ECHO "Mx G2000: Error : "
   $_ECHO "          Log directory :$LOG_PATH does not exist !"
   $_ECHO "          Please create it."
   exit 1
fi
if [ ! -w $LOG_PATH ] ; then
   $_ECHO "Mx G2000: Error : "
   $_ECHO "          Log directory :$LOG_PATH does not have good rights !"
   $_ECHO "          Please add write permission."
   exit 1
fi
if [ -f $LOG_PATH/$LOG_FILE.pid ] ; then
  $_ECHO "The service may already run, please check below."
  Process_Status
#  exit 1
  if [ -f $LOG_PATH/$LOG_FILE.pid ] ; then
        $_ECHO "\nThe service already run."
        exit 1
  else
        $_ECHO "\nCleanup done \nLaunching Process" 
  fi
fi

if [ $APPEND_LOG = 1 ] ; then
        $_ECHO "---------------\n" >> $LOG_PATH/$LOG_FILE.log
else 
        $_ECHO "---------------\n" > $LOG_PATH/$LOG_FILE.log
fi
$_ECHO "Start time `date` by $USER_NAME\n" >> $LOG_PATH/$LOG_FILE.log
$_ECHO "File descriptors raised to `ulimit -n` for current cmd.\n" >> $LOG_PATH/$LOG_FILE.log
$_ECHO "Java option used : $JVM_OPTION " >>$LOG_PATH/$LOG_FILE.log
java $JVM_OPTION -version 2>&1 | $_TEE -a $LOG_PATH/$LOG_FILE.log
$_ECHO "" >>$LOG_PATH/$LOG_FILE.log
$_ECHO "" 
}

Update_Log_Pid_Files() {
#Param 1 : The process ID number  $! 
#Param 2 : If the process is a shell exec 1 else 0

sleep 1 #Needed to give time to the process to defunct.

PID_NB=
if [ "$OS_TYPE" = "Linux" ]; then
   _PID_NB=
   PTEE_PID_NB=
   PTEE_PID_NB=`$_PS -eaf | $_AWK ' \$2 == '$!' ' | $_AWK '{ print \$3 }'`
# DSLECOMTE-DEF0023546-REMOTE_SHELL_LINUX
   _PID_NB=`$_PS -eaf | $_AWK ' \$3 == '$PTEE_PID_NB' ' | $_AWK ' \$2 != '$!' '`
# FSLECOMTE-DEF0023546-REMOTE_SHELL_LINUX
   PID_NB=`echo $_PID_NB | $_AWK '{ print \$2 }'`
else 
   PID_NB=`$_PS -eaf | $_AWK ' \$3 == '$!' ' | $_AWK '{ print \$2 }'`
fi

if [ $2 = 1 ] ; then
   PID_NB=`$_PS -eaf | $_AWK ' \$3 == '$PID_NB' ' | $_AWK '{ print \$2 }'`
fi
if [ "$PID_NB" = "" ] ; then
   $_ECHO "\n"
   $_ECHO "Mx G2000: Fatal ERROR: "
   $_ECHO "          The service did not start."
   $_ECHO "          See messages on your screen or on $LOG_PATH/$LOG_FILE.log file."
   $_ECHO "\nAt `date`\n" >> $LOG_PATH/$LOG_FILE.log
   $_ECHO "  Service failed to start (message above)\n" >> $LOG_PATH/$LOG_FILE.log
   $_ECHO "\n---------------\n" >> $LOG_PATH/$LOG_FILE.log
   if [ -f $LOG_PATH/$LOG_FILE.pid ] ; then
      $_ECHO "WARNING !!\n The service may be already launched.\n"  | $_TEE -a $LOG_PATH/$LOG_FILE.log
   fi 
else
   $_ECHO "\n***\nPID:$PID_NB\n***\n" | $_TEE -a $LOG_PATH/$LOG_FILE.log
   $_ECHO "Logging stdout and stderr to $LOG_PATH/$LOG_FILE.log\n\n" 
   $_ECHO $PID_NB > $LOG_PATH/$LOG_FILE.pid
fi
}

Stop_Service() {
# Params : ID of Service 
Define_Log_File_Name $1
if [ ! -f $LOG_PATH/$LOG_FILE.pid ] ; then
   $_ECHO "Service $1 doesn't seem's to run !"
   exit 1
fi
for file in  `$_LS $LOG_PATH/$LOG_FILE.pid`
do
        FILE_OWNER=`$_LS_L $file | $_AWK '{print $3}'`
        if [ "$FILE_OWNER" != "$USER_NAME" ]; then
           $_ECHO " Not owner of $LOG_PATH/$LOG_FILE.pid"
           $_ECHO " Service not Stopped."
        else
           KILL_PID=`cat $file`
           $_ECHO "Found process pid $KILL_PID file `basename $file` "
           kill -9 $KILL_PID >/dev/null
           if [ $? -eq 0 ] ; then 
              $_ECHO "***************" >> $LOG_PATH/$LOG_FILE.log
              $_ECHO "Service stopped at `date` by $USER_NAME" | $_TEE -a $LOG_PATH/$LOG_FILE.log
              $_ECHO "***************\n" >> $LOG_PATH/$LOG_FILE.log
              rm $file
           else 
              $_ECHO "Process ID not found"
              rm $file
           fi
        fi
done

}

Kill_All() {
if [ ! -f $LOG_PATH/*.pid ] ; then
   $_ECHO "No Service running."
   exit 0
else
   for file in `$_LS $LOG_PATH/${MACHINE_NAME}*.pid`
      do
        FILE_OWNER=`$_LS_L $file | $_AWK '{print $3}'`
        if [ "$FILE_OWNER" != "$USER_NAME" ]; then
           $_ECHO " Not owner of $LOG_PATH/$file"
           $_ECHO " Service not Stopped."
        else
           LOG_FILE=`$_LS $file | sed 's/.pid//'`
           SERVICE=`echo $file | cut -d"." -f2`
           KILL_PID=`cat $file`
           $_ECHO "Found process pid $KILL_PID file `basename $file` "
           kill -9 $KILL_PID >/dev/null
           if [ $? -eq 0 ] ; then
              $_ECHO "***************" >> $LOG_FILE.log
              $_ECHO "Service stopped at `date` by $USER_NAME" | $_TEE -a $LOG_FILE.log
              $_ECHO "***************\n" >> $LOG_FILE.log
              rm $file
           else
              $_ECHO "Process ID not found, assuming process is dead"
              rm $file
           fi
        fi
      done
fi
}

Process_Status() {
# Params : None

if [ ! -f $LOG_PATH/*.pid ] ; then
   $_ECHO "No Service running."
   exit 0
fi

$_ECHO "\nFound running service(s) :"
for files in `$_LS $LOG_PATH/*.pid`
do
        SERVICE=`echo $files | sed s/\.pid//`
        SERVICE=`basename $SERVICE`
        SERVICE_LOCATION=`echo  $SERVICE | cut -d"." -f1`
        if [  "$SERVICE_LOCATION" = "$MACHINE_NAME" ] ; then
            PID_NB=`cat $files`
            # FOUND=`$_PS -eaf | $_AWK ' \$2 == '$PID_NB' '`
            FOUND=`$_PS -fp $PID_NB | grep $PID_NB`
            if [ ! "$FOUND" = "" ] ; then
            INFOS=`echo $FOUND | $_AWK '{ if ( \$5 ~ /:/ ) {  print " UID: "\$1 " PID: "\$2  " CPUTIME: "\$7 " STIME: "\$5 } else { print " UID: "\$1 " PID: "\$2  " CPUTIME: "\$8 " STIME: "\$5" " \$6 } }'` 
                   $_ECHO " $SERVICE infos : \n\t$INFOS"
            else
                   $_ECHO " $SERVICE not running, removing PID file." 
                   rm $files
                fi
        else
            $_ECHO " $SERVICE infos : \n\tService located on $SERVICE_LOCATION\n\tRun status from $SERVICE_LOCATION"
        fi 

done

} 

help() {
Setting_Env
# Params : None
        cat <<END_OF_HELP | more

$0 usage:
Version : $MAJOR_VERSION.$MINOR_VERSION

`basename $0` [ -option ] [ Param ]* [ -k | -killall ]

Options:  -i:file               : use file as the setting.
          -fs | -filserver      : launch file server service.
          -xmls | -xmlserver    : launch xmlserver server service.
          -xmlsnh | -xmlsnohub  : launch xmlserver server service without a hub.
          -hub                  : launch hub service.
          -tm                   : launch transactionmanager server service.
          -l | -launcher        : launch launcher server service.
          -mxml | -mxmlex       : launch mxmlexchange server service.
          -mxnet | -murexnet    : launch murexnet server service.
          -olk | -import        : launch olk import server service.
          -mxp | -mxparam       : launch mxparam server service.
          -rtisession | -rtimportsession : launch a session of rtimport server service.
          -rticachesession | -rtimportcachesession : launch a session of rtimport-cache server service.
          -rticache | -rtimportcache      : launch rtimport cache server service
          -rtifxgsession | -rtimportfixingsession : launch a session of rtimport fixing server service.
          -rti | -rtimport      : launch rtimport server service
          -rtifxg | -rtimportfixing : launch rtifixing import server service.
          -rtiexportresults     : launch rtimport export results service
          -mxcontrib | -mpcs    : launch mxcontribution server service.  
          -feeder               : launch activity feeder server service. 
          -mdcs | -cache        : launch MDCS cache service.
          -mdrs                 : launch MarketData repository service
          -client | -mx         : launch mx client.
          -clientmacro | -mxmacro : launch mx client in macro mode, use 
                                    /MXJ_SCRIPT_READ_FROM:script.xml to 
                                    change default script file.
          -monit | -monitor     : launch monitor.
          -smonit | -smonitor   : launch monitor in script mode (need 
                                  extra arg /MXJ_CONFIG_FILE:script_file.xml).
          -mxrdt                : remote diagnostic tool
          -xmlreq | -xmlrequest : launch xmlRequestScript class (need 
                                  extra arg /MXJ_CONFIG_FILE:xmlRequestScript.xml).
          -p | -password        : launch password encryption.
          
          -k | -kill            : option to stop service.
          -killall              : stop all running services.
          -s | -status [-loop]  : show services status [every $LOOP_TIME sec].
 
          -j:[java option] | -jopt:[java option]  : add a JVM option, can be used
                                                    as many times as options needed.
          -h | -help            : this help.

        To stop a service use the same param as the start and add -k
          ex :  to stop $0 -fs 
                   use  $0 -fs -k  
                to stop $0 -l /MXJ_CONFIG_FILE:mylauncher.mxres
                   use  $0 -l -k /MXJ_CONFIG_FILE:mylauncher.mxres
                use $0 -s to see running services.
                use $0 -killall to stop all running services.
Param used : 
        Setting File:$SETTINGS_FILE
        /MXJ_FILESERVER_HOST:$MXJ_FILESERVER_HOST
        /MXJ_FILESERVER_PORT:$MXJ_FILESERVER_PORT
        /MXJ_JAR_FILE:$MXJ_JAR_FILE
        /MXJ_PORT:$MXJ_PORT (set for backward comatibility)
        /MXJ_HOST:$MXJ_HOST (set for backward comatibility)
        /MXJ_SITE_NAME:$MXJ_SITE_NAME
        /MXJ_HUB_NAME:$MXJ_HUB_NAME
        /MXJ_PLATFORM_NAME:$MXJ_PLATFORM_NAME
        /MXJ_PROCESS_NICK_NAME:$MXJ_PROCESS_NICK_NAME
        /MXJ_CONFIG_FILE:$MXJ_CONFIG_FILE
        /MXJ_MXMLEX_CONFIG_FILE:$MXJ_MXMLEX_CONFIG_FILE
        /MXJ_CONTRIBUTION_CONFIG_FILE:$MXJ_CONTRIBUTION_CONFIG_FILE
        /MXJ_LOG_LEVEL:$MXJ_LOG_LEVEL
        /MXJ_SCRIPT_READ_FROM:$MXJ_SCRIPT
        MUREXNET_PORT:$MUREXNET_PORT
Specific $SETTINGS_FILE settings:
        File Server            :$FILESERVER_ARGS
        XmlServer args         :$XML_SERVER_ARGS
        Hub args               :$HUB_HOME_ARGS
        MxMlExchange args      :$MXML_SERVER_ARGS
        Launcher args          :$LAUNCHER_ARGS
        Murexnet args          :$MUREXNET_ARGS
        Real Time host display :$RTISESSION_XWIN_DISP
		MDCS args              :$MDCS_ARGS

Environment:
        LOG_PATH:$LOG_PATH
        APPEND_LOG:$APPEND_LOG
        JAVAHOME:$JAVAHOME
        SYBASE:$SYBASE
        SYBASE_OCS:$SYBASE_OCS
        ORACLE_HOME:$ORACLE_HOME

END_OF_HELP
$_ECHO "\nFile descriptors raised to `ulimit -n` for current shell.\n"
case $OS_TYPE in
        SunOS )
        $_ECHO "\nLD_LIBRARY_PATH=$LD_LIBRARY_PATH\n"
        ;;
        AIX )
        $_ECHO "\nLIBPATH=$LIBPATH\n"
        ;;
        HP-UX )
        $_ECHO "\nSHLIB_PATH=$SHLIB_PATH\n"
        ;;
        Linux )
        $_ECHO "\nLD_LIBRARY_PATH=$LD_LIBRARY_PATH\n"
        ;;
        * )
        $_ECHO "Warning : Do not know how to handle this OS type $OS_TYPE."
        ;;
esac

$_ECHO `java -version`
if [ "$SYBASE" != "" ] ; then
   if [ ! -f $SYBASE/$SYBASE_OCS/bin/isql ] ; then
      $_ECHO "Mx G2000: isql not found, please check the SYBASE environment variable"
      $_ECHO "          in the $SETTINGS_FILE script file"
   else
      $SYBASE/$SYBASE_OCS/bin/isql -v
   fi
fi
if [ "$ORACLE_HOME" != "" ] ; then
   if [ ! -f $ORACLE_HOME/bin/sqlplus ] ; then
      $_ECHO "Mx G2000: sqlplus not found, please check the ORACLE_HOME environment variable"
      $_ECHO "          in the $SETTINGS_FILE script file"
   else
      $ORACLE_HOME/bin/sqlplus -V
   fi
fi
exit 0
}

getParams() {
while [ $# != 0 ]
        do
        ARG0=$1

        PARAM=`echo $ARG0 | cut -f1 -d":"`
        VALUE=`echo $ARG0 | cut -f2- -d":"`

        if [ "$VALUE" = "" ] ; then
           help
           exit 0
        fi
        case $PARAM in
            -help | /help | -h | /h | -env | /env)
                help
                exit 0
                ;;
            -i )
                SETTINGS_FILE=$VALUE
                if [ $# -eq 1  ] ; then
                   help
                   exit 0
                fi
                ;;
            -xmls | -xmlserver )
                XMLS=1
                ;;
            -xmlsnohub | -xmlservernohub | -xmlsnh )
                XMLSNOHUB=1
                ;;
            -hub )
                HUB=1
                ;;
            -tm )
                TM=1
                ;;
            -fs | -filserver )
                FS=1
                ;;
            -l | -launcher )
                LAUNCHER=1
                ;;
            -mxml | -mxmlex )
                MXMLEX=1
                ;;
            -mdcs | -cache )
                MDCS=1
                ;;
            -mdrs )
                MDRS=1
                ;;
            -olk | -import ) 
                OLK=1
                ;;
            -mxp | -mxparam )
                MXPARAM=1
                ;;
            -rtisession | -rtimportsession )
                RTISESSION=1
                ;;
            -rticachesession | -rtimportcachesession )
                RTICACHESESSION=1
                ;;
            -rtifxgsession | -rtimportfixingsession )
                RTIFXGSESSION=1
                ;;
            -rti | -rtimport )
                RTIMPORT=1
                ;;
            -rticache | -rtimportcache )
                RTIMPORTCACHE=1
                ;;
            -rtifxg | -rtimportfixing )
                RTIFXG=1
                ;;
            -rtiexportresults )
                RTIEXPORTRESULTS=1
                ;;                
            -mxcontrib | -mpcs )
                MXCONTRIB=1
                ;;
            -feeder )
                FEEDER=1
                ;;
            -mxnet | -murexnet )
                MUREXNET=1
                ;;
            -client | -mx )
                CLIENT=1
                ;;
            -clientmacro | -mxmacro )
                CLIENTMACRO=1
                ;;
            -monit | -monitor )
                MONIT=1
                ;;
            -smonit | -smonitor )
                MXJ_CONFIG_FILE=""
                export MXJ_CONFIG_FILE
                S_MONIT=1
                ;;
            -mxrdt)
                MXRDT=1
                ;;
            -xmlreq | -xmlrequest )
                MXJ_CONFIG_FILE=""
                export MXJ_CONFIG_FILE
                XMLREQ=1
                ;;
            -p | -password )
                PASSWORD=1
                ;;
            -stop | stop | -kill | kill | -k | k )
                STOP=1
                ;;
            -stopall | stopall | -killall | killall )
                STOPALL=1
                ;;
            -status | status | -s | s )
                STATUS=1
                ;;
            -loop )
                LOOP=1
                ;;
            -j | j | -jopt | jopt )
                JVM_OPTION=$JVM_OPTION" "$VALUE
                $_ECHO "Using JVM option:$VALUE "
                ;;
            /MXJ_PORT | -MXJ_PORT )
                MXJ_PORT=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_HOST | -MXJ_HOST )
                MXJ_HOST=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_SITE_NAME | -MXJ_SITE_NAME )
                MXJ_SITE_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_HUB_NAME | -MXJ_HUB_NAME )
                MXJ_HUB_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_FILESERVER_HOST | -MXJ_FILESERVER_HOST )
                MXJ_FILESERVER_HOST=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_FILESERVER_PORT | -MXJ_FILESERVER_PORT )
                MXJ_FILESERVER_PORT=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_JAR_FILE | -MXJ_JAR_FILE )
                MXJ_JAR_FILE=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_GUI_JAR_FILE | -MXJ_GUI_JAR_FILE )
                MXJ_GUI_JAR_FILE=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_CONFIG_FILE | -MXJ_CONFIG_FILE )
                NBP=`echo $VALUE | $_AWK -F. '{ print NF }'`
                if [ "$NBP" = "2" ] && [ "$LAUNCHER" = "1" ] ; then
                   VALUE=`echo $MXJ_CONFIG_FILE | $_AWK  -F. '{ print \$1"."\$2"."\$3"." }'`$VALUE
                   echo "Guessing config file is $VALUE"
                fi 
                if [ "$NBP" = "1" ] && [ "$LAUNCHER" = "1" ] ; then
                   VALUE=`echo $MXJ_CONFIG_FILE | $_AWK  -F. '{ print \$1"."\$2"."\$3"." }'`${VALUE}.mxres
                   echo "Guessing config file is $VALUE"
                fi
                MXJ_CONFIG_FILE=$VALUE
                MXJ_MXMLEX_CONFIG_FILE=$VALUE 
                MXJ_CONTRIBUTION_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_MXMLEX_CONFIG_FILE | -MXJ_MXMLEX_CONFIG_FILE )
                MXJ_MXMLEX_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_MDCS_CONFIG_FILE | -MXJ_MDCS_CONFIG_FILE )
                MXJ_MDCS_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_MDRS_CONFIG_FILE | -MXJ_MDRS_CONFIG_FILE )
                MXJ_MDRS_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_CONTRIBUTION_CONFIG_FILE | MXJ_CONTRIBUTION_CONFIG_FILE )
                MXJ_CONTRIBUTION_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_ACTIVITY_FEEDER_CONFIG_FILE | MXJ_ACTIVITY_FEEDER_CONFIG_FILE )
                MXJ_ACTIVITY_FEEDER_CONFIG_FILE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_PLATFORM_NAME | -MXJ_PLATFORM_NAME )
                MXJ_PLATFORM_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_PROCESS_NICK_NAME | -MXJ_PROCESS_NICK_NAME )
                MXJ_PROCESS_NICK_NAME=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_LOG_LEVEL| -MXJ_LOG_LEVEL)
                MXJ_LOG_LEVEL=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_POLICY_FILE | -MXJ_POLICY_FILE )
                MXJ_POLICY_FILE=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_SCRIPT_READ_FROM| -MXJ_SCRIPT_READ_FROM)
                MXJ_SCRIPT=$VALUE 
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXJ_INSTALLATION_CODE| -MXJ_INSTALLATION_CODE )
                MXJ_INSTALLATION_CODE=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                EXTRA_ARGS="$EXTRA_ARGS $ARG0"
                ;;
            /MUREXNET_PORT| -MUREXNET_PORT| MUREXNET_PORT)
                MUREXNET_PORT=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;
            /MXRDT_OUTPUT_FILENAME| -MXRDT_OUTPUT_FILENAME)
                MXRDT_OUTPUT_FILENAME=$VALUE
                $_ECHO "Using $PARAM:$VALUE"
                ;;

            * )
                EXTRA_ARGS="$EXTRA_ARGS $ARG0"
                ;;
        esac
        shift
done
}

#Main

XMLS=0
XMLSNOHUB=0
HUB=0
TM=0
FS=0
LAUNCHER=0
MXMLEX=0
OLK=0
MDCS=0
MDRS=0
MXPARAM=0
RTISESSION=0
RTICACHESESSION=0
RTIFXGSESSION=0
RTIMPORT=0
RTIMPORTCACHE=0
RTIFXG=0
RTIEXPORTRESULTS=0
MXCONTRIB=0
FEEDER=0
MUREXNET=0
CLIENT=0
CLIENTMACRO=0
MONIT=0
S_MONIT=0
MXRDT=0
PASSWORD=0
STOP=0
STOPALL=0
STATUS=0
LOOP=0
XMLREQ=0
JVM_OPTION="-Xmx512M"

if [ "$OS_TYPE" = "AIX" ]; then
  JVM_OPTION=$JVM_OPTION" -Djavax.xml.parsers.SAXParserFactory=org.apache.xerces.jaxp.SAXParserFactoryImpl "
  JVM_OPTION=$JVM_OPTION" -Djavax.xml.parsers.DocumentBuilderFactory=org.apache.xerces.jaxp.DocumentBuilderFactoryImpl "
  JVM_OPTION=$JVM_OPTION" -Djavax.xml.transform.TransformerFactory=org.apache.xalan.processor.TransformerFactoryImpl "
fi

MXJ_INSTALLATION_CODE=""
EXTRA_ARGS=

if [ $# = 0 ] ; then
        help;
        exit 0 ;
fi

Setting_Env
getParams $*

if [ "$EXTRA_ARGS" != "" ] ; then
        $_ECHO "Extra arguments used: $EXTRA_ARGS"
fi

if [ $FS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service fileserver 
        else
                Fileserver $*;
        fi
fi
if [ $TM = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service transactionmanager
        else
                TransactionManager $*;
        fi
fi
if [ $XMLS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service xmlserver
        else
                Xmlserver $*;
        fi
fi
if [ $XMLSNOHUB = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service xmlservernohub
        else
                XmlserverNoHub $*;
        fi
fi

if [ $HUB = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service hubhome
        else
                HubHome $*;
        fi
fi

if [ $LAUNCHER = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service launcher
        else
                Launcher $*;
        fi
fi
if [ $MXMLEX = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mxmlexchange
                Stop_Service mxmlexchangesecondary
                Stop_Service mxmlexchangespaces
        else
                Mxmlexchange $*;
        fi
fi
if [ $MDCS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mdcs
        else
                MDCS_CACHE $*;
        fi
fi
if [ $MDRS = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mdrs
        else
                MDRS_ENGINE $*;
        fi
fi
if [ $OLK = 1 ] ; then
        if [ $STOP = 1 ] ; then
               Stop_Service olk
        else
                Olk $*;
        fi
fi
if [ $MXPARAM = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service mxparam
                MxParam stop $*;
        else
                MxParam start $*;
        fi
fi
if [ $RTISESSION = 1 ] ; then
                RtImport session $*;
fi
if [ $RTICACHESESSION = 1 ] ; then
                RtImport rticachesession $*;
fi
if [ $RTIFXGSESSION = 1 ] ; then
                RtImport fxgsession $*;
fi

if [ $RTIMPORT = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service rtimport
                RtImport stop $*;
        else
                RtImport start $*;
        fi
fi
if [ $RTIMPORTCACHE = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service rtimportcache
                RtImport rticachestop $*;
        else
                RtImport rticachestart $*;
        fi
fi
if [ $RTIEXPORTRESULTS = 1 ] ; then
                RtImport rtiexportresults $*;
fi

if [ $RTIFXG = 1 ] ; then
        if [ $STOP = 1 ] ; then
                #Stop_Service rtimport
                RtImport fxgstop $*;
        else
                RtImport fxgstart $*;
        fi
fi
if [ $MXCONTRIB = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service mxcontrib
        else
                MxContribution $*;
        fi
fi
if [ $FEEDER = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service feeder
        else
                MxActivityFeeder $*;
        fi
fi
if [ $MUREXNET = 1 ] ; then
        if [ $STOP = 1 ] ; then
                Stop_Service murexnet
        else
                Murexnet $*;
        fi
fi
if [ $CLIENT = 1 ] ; then
        Client $*;
fi
if [ $CLIENTMACRO = 1 ] ; then
        ClientMacro $*;
fi
if [ $MONIT = 1 ] ; then
        Monitor $*;
fi
if [ $S_MONIT = 1 ] ; then
        Script_Monitor $*;
fi
if [ $MXRDT = 1 ] ; then
        MxRdt
fi
if [ $XMLREQ = 1 ] ; then
        XmlRequestScript $*;
fi
if [ $PASSWORD = 1 ] ; then
        PasswordEncryption $*;
fi
if [ $STATUS = 1 ] ; then

        Process_Status $*;
        while [ $LOOP = 1 ] 
        do
                $_ECHO "\nSleeping $LOOP_TIME sec"
                sleep $LOOP_TIME
                Process_Status $*
        done
fi
if [ $STOPALL = 1 ] ; then
        Kill_All
fi
#END of SCRIPT
