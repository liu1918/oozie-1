

[::Go back to Oozie Documentation Index::](index.html)

-----

# Oozie Ssh Action Extension

<!-- MACRO{toc|fromDepth=1|toDepth=4} -->

## Ssh Action

The `ssh` action starts a shell command on a remote machine as a remote secure shell in background. The workflow job
will wait until the remote shell command completes before continuing to the next action.

The shell command must be present in the remote machine and it must be available for execution via the command path.

The shell command is executed in the home directory of the specified user in the remote host.

The output (STDOUT) of the ssh job can be made available to the workflow job after the ssh job ends. This information
could be used from within decision nodes. If the output of the ssh job is made available to the workflow job the shell
command must follow the following requirements:

   * The format of the output must be a valid Java Properties file.
   * The size of the output must not exceed 2KB.

Note: Ssh Action will fail if any output is written to standard error / output upon login (e.g. .bashrc of the remote
user contains ls -a).

Note: Ssh Action will fail if oozie fails to ssh connect to host for action status check
(e.g., the host is under heavy load, or network is bad) after a configurable number (3 by default) of retries.
The first retry will wait a configurable period of time ( 3 seconds by default) before check.
The following retries will wait 2 times of previous wait time.

Note: [OOZIE-3569](https://issues.apache.org/jira/browse/OOZIE-3569)  patch fixes the bug that
SSH Action will be incorrectly determined to be successful when Oozie pid is killed externally.
It's released in version 5.3.0.
Please note that this is an incompatible change and may cause some SSH Action tasks to fail during updating.

**Syntax:**


```
<workflow-app name="[WF-DEF-NAME]" xmlns="uri:oozie:workflow:1.0">
    ...
    <action name="[NODE-NAME]">
        <ssh xmlns="uri:oozie:ssh-action:0.1">
            <host>[USER]@[HOST]</host>
            <command>[SHELL]</command>
            <args>[ARGUMENTS]</args>
            ...
            <capture-output/>
        </ssh>
        <ok to="[NODE-NAME]"/>
        <error to="[NODE-NAME]"/>
    </action>
    ...
</workflow-app>
```

The `host` indicates the user and host where the shell will be executed.

**IMPORTANT:** The `oozie.action.ssh.allow.user.at.host` property, in the `oozie-site.xml` configuration, indicates if
an alternate user than the one submitting the job can be used for the ssh invocation. By default this property is set
to `true`.

The `command` element indicates the shell command to execute.

The `args` element, if present, contains parameters to be passed to the shell command. If more than one `args` element
is present they are concatenated in order. When an `args` element contains a space, even when quoted, it will be considered as
separate arguments (i.e. "Hello World" becomes "Hello" and "World").  Starting with ssh schema 0.2, you can use the `arg` element
(note that this is different than the `args` element) to specify arguments that have a space in them (i.e. "Hello World" is
preserved as "Hello World").  You can use either `args` elements, `arg` elements, or neither; but not both in the same action.

If the `capture-output` element is present, it indicates Oozie to capture output of the STDOUT of the ssh command
execution. The ssh command output must be in Java Properties file format and it must not exceed 2KB. From within the
workflow definition, the output of an ssh action node is accessible via the =String action:output(String node,
String key)= function (Refer to section '4.2.6 Action EL Functions').

The configuration of the `ssh` action can be parameterized (templatized) using EL expressions.

**Example:**


```
<workflow-app name="sample-wf" xmlns="uri:oozie:workflow:1.0">
    ...
    <action name="myssjob">
        <ssh xmlns="uri:oozie:ssh-action:0.1">
            <host>foo@bar.com<host>
            <command>uploaddata</command>
            <args>jdbc:derby://bar.com:1527/myDB</args>
            <args>hdfs://foobar.com:8020/usr/tucu/myData</args>
        </ssh>
        <ok to="myotherjob"/>
        <error to="errorcleanup"/>
    </action>
    ...
</workflow-app>
```

In the above example, the `uploaddata` shell command is executed with two arguments, `jdbc:derby://foo.com:1527/myDB`
and `hdfs://foobar.com:8020/usr/tucu/myData`.

The `uploaddata` shell must be available in the remote host and available in the command path.

The output of the command will be ignored because the `capture-output` element is not present.

## Appendix, Ssh XML-Schema

### AE.A Appendix A, Ssh XML-Schema

#### Ssh Action Schema Version 0.2


```
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:ssh="uri:oozie:ssh-action:0.2" elementFormDefault="qualified"
           targetNamespace="uri:oozie:ssh-action:0.2">
.
    <xs:element name="ssh" type="ssh:ACTION"/>
.
    <xs:complexType name="ACTION">
        <xs:sequence>
            <xs:element name="host" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="command" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:choice>
              <xs:element name="args" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
              <xs:element name="arg" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            </xs:choice>
            <xs:element name="capture-output" type="ssh:FLAG" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>
.
    <xs:complexType name="FLAG"/>
.
</xs:schema>
```

#### Ssh Action Schema Version 0.1


```
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:ssh="uri:oozie:ssh-action:0.1" elementFormDefault="qualified"
           targetNamespace="uri:oozie:ssh-action:0.1">
.
    <xs:element name="ssh" type="ssh:ACTION"/>
.
    <xs:complexType name="ACTION">
        <xs:sequence>
            <xs:element name="host" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="command" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="args" type="xs:string" minOccurs="0" maxOccurs="unbounded"/>
            <xs:element name="capture-output" type="ssh:FLAG" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>
.
    <xs:complexType name="FLAG"/>
.
</xs:schema>
```

[::Go back to Oozie Documentation Index::](index.html)


