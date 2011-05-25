## Oozie Exceptions
### Error Msg:<br>
Caused by: org.apache.oozie.service.HadoopAccessorException: E0902: Exception occured: [java.io.IOException: Call to localhost/127.0.0.1:54310 failed on local exception: java.io.EOFException]
	at org.apache.oozie.service.KerberosHadoopAccessorService.createFileSystem(KerberosHadoopAccessorService.java:208)
	at org.apache.oozie.service.AuthorizationService.authorizeForApp(AuthorizationService.java:285)
### Solution:
Copy/Paste oozie.services property tag set from oozie-default.xml to oozie-site.xml. Remove _org.apache.oozie.service.KerberosHadoopAccessorService_.

### Error Msg:
Error: HTTP error code: 500 : Internal Server Error
Error: E0902 : E0902: Exception occured: [java.io.IOException: Call to localhost/127.0.0.1:54310 failed on local exception: java.io.EOFException]
### Solution:
Add _org.apache.oozie.service.HadoopAccessorService_ in oozie.services property tag in oozie-site.xml.
