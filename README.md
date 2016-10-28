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



