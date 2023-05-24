# ace-user-variable-examples
Examples of using ACE user variables in message flows

## ESQLApplication

This application shows ESQL reading the value of two user variables from ESQL by declaring them as
"EXTERNAL" in the same way as is done for user-defined properties. In this case, the properties are
not overridden anywhere else (such as with mqsiapplybaroverride) and so the ESQL runtime looks for
matching user variables.

### Initial deploy and test 

The application can be deployed and run in an independent IntegrationServer or using a node-associated 
server, and the two flows should start automatically. The timer-based flow is most useful for 
independent servers, as it prints the user variable values to the console using a trace node:

```
tdolby@IBM-PF3K066L:/$ IntegrationServer -w ~/tmp/ace-user-variable-examples-work-dir
2023-05-24 09:00:27.723216: BIP1990I: Integration server 'ace-user-variable-examples-work-dir' starting initialization; version '12.0.8.0' (64-bit)
2023-05-24 09:00:27.728974: BIP9905I: Initializing resource managers.
2023-05-24 09:00:29.683948: BIP9906I: Reading deployed resources.
2023-05-24 09:00:29.686588: BIP9907I: Initializing deployed resources.
2023-05-24 09:00:29.687742: BIP2155I: About to 'Initialize' the deployed resource 'ESQLApplication' of type 'Application'.
2023-05-24 09:00:29.817392: BIP2155I: About to 'Start' the deployed resource 'ESQLApplication' of type 'Application'.
An http endpoint was registered on port '7800', path '/ESQLApplication'.
2023-05-24 09:00:29.882952: BIP3132I: The HTTP Listener has started listening on port '7800' for 'http' connections.
2023-05-24 09:00:29.883084: BIP1996I: Listening on HTTP URL '/ESQLApplication'.
Started native listener for HTTP input node on port 7800 for URL /ESQLApplication
2023-05-24 09:00:29.883320: BIP2269I: Deployed resource 'HTTPFlow' (uuid='HTTPFlow',type='MessageFlow') started successfully.
2023-05-24 09:00:29.883494: BIP2269I: Deployed resource 'TimerFlow' (uuid='TimerFlow',type='MessageFlow') started successfully.
2023-05-24 09:00:29.883624: BIP9332I: Application 'ESQLApplication' has been reloaded successfully.
2023-05-24 09:00:29.884332: BIP3051E: Error message 'Root.JSON:
( ['json' : 0x7f4e8005b650]
  (0x01000000:Object):Data = (
    (0x03000000:NameValue):firstUserVariable  = 'valueFromServerConfYaml' (CHARACTER)
    (0x03000000:NameValue):secondUserVariable = 'valueFromSECOND_USER_VARIABLE:' (CHARACTER)
  )
)
' from trace node 'TimerFlow.Trace'.
2023-05-24 09:00:30.117236: BIP2866I: IBM App Connect Enterprise administration security is inactive.
2023-05-24 09:00:30.127390: BIP3132I: The HTTP Listener has started listening on port '7600' for 'RestAdmin http' connections.
2023-05-24 09:00:30.133004: BIP1991I: Integration server has finished initialization.
```

The HTTP-based flow returns the user variable values as an HTTP reply:
```
tdolby@IBM-PF3K066L:/$ curl http://localhost:7800/ESQLApplication
{"firstUserVariable":"valueFromServerConfYaml","secondUserVariable":"valueFromSECOND_USER_VARIABLE:"}
```

### Updating the user variables

The user variables can be changed from the independent server command line
```
IntegrationServer -w ~/tmp/ace-user-variable-examples-work-dir --user-firstUserVariable test 
```
or by setting the variables in server.conf.yaml:
```
UserVariables:
  firstUserVariable: 'valueFromServerConfYaml'
  secondUserVariable: 'otherValueFromServerConfYaml'
```
The file [ESQLApplication/example-overrides-server.conf.yaml](ESQLApplication/example-overrides-server.conf.yaml) can
be copied into the server work directory as `overrides/server.conf.yaml` as a starting point. The example also
reads an environment variable as part of setting `secondUserVariable` to allow further configurability.

### Other notes

The [ESQL code](ESQLApplication/Flow_Compute.esql) declares the variables with a default value to 
avoid failures on startup:
```
DECLARE firstUserVariable EXTERNAL CHARACTER 'defaultValue';
DECLARE secondUserVariable EXTERNAL CHARACTER 'defaultValue';
```
The default is not required, but if it not present then the application will not start.
