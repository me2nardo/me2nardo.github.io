---
layout: post
title:  "Tomcat production settings"
date:   2017-06-26 16:31:01 
categories: tomcat
---
For NIO, NIO2, APR 
`acceptCount` - The maximum queue length for incoming connection requests when all possible request processing threads are in use. Any requests received when the queue is full will be refused. The default value is 100. Increase to 400.

`compression` - set to ON. GZIP compression in an attempt to save server bandwidth

`compressionMinSize` - set comprasion min size default 2048 kb.

`executor` - also we can define custom executor with custom settings.

`connectionTimeout` - time to accept connection

`connectionUploadTimeout` - Specifies the timeout, in milliseconds, to use while a data upload is in progress. 

`keepAliveTimeout` - Set keep alive. Or it will get default os value.

`protocol` - Sets the protocol to handle incoming traffic. The default value is HTTP/1.1 which uses an auto-switching mechanism to select either a blocking Java based connector or an APR/native based connector. If the PATH (Windows) or LD_LIBRARY_PATH (on most unix systems) environment variables contain the Tomcat native library, the APR/native connector will be used. If the native library cannot be found, the blocking Java based connector will be used. Note that the APR/native connector has different settings for HTTPS than the Java connectors. In my case org.apache.coyote.http11.Http11NioProtocol.
