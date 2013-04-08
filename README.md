CodeSamples_Logback-AMQP
========================
This code uses messaging to elegantly solve a common problem with portable application frameworks. It and its use are best described by the documentation I wrote for it on the project wiki. I've pasted it below.

----------------------------------------

The Iris application framework is designed to be flexible and able to run in different environments. For example, Iris needs to run at NIDS as well as the Telecommunications Gateway. Each of these environments may have their own way of monitoring the applications that run within them and they need to be able to monitor the Iris framework and it's applications. One aspect of monitoring is examining significant events during the runtime of an application through that application's log data. The Iris framework applications may be numerous as produce many log files in different locations and on different servers. To avoid putting environment-specific monitoring code into portable Iris applications and to avoid attempting to track down and monitor various Iris application's log files, all Iris applications publish their log data to a single AMQP topic exchange on the Iris message broker. Consumers of this data get the log information pushed to them in real time and are able to filter which log data to receive based on the originating application and log level, or severity. This allows system administrators of a particular Iris installation to write an Iris log data consumer in a language of their choice, subscribe to only the data important to them, and route that data to their environment's monitoring systems.

Of course, writing an Iris log data consumer requires a little knowledge of AMQP, but it's pretty simple. All the resources you need to get started in a language of your choice are available at the RabbitMQ Clients & Developer Tools page.

All Iris log data is published to a single topic exchange called logExchange. Your client will create a queue for itself and bind that queue to this exchange.

Iris log data is published to the logExchange using a hierarchical routing key containing two parts, the first for the logging context (this usually corresponds to the originating application) and the second for the logging level, or severity. For example, a log message coming from IrisDecoders with a level of ERROR would be published with a routing key of IrisDecoders.ERROR.

Currently, the logging contexts in use are:

	IrisTextIngest
	IrisDecoders
	IrisJsonRPC
	HazCollectEx

The possible log levels, or severities, in increasing order are:

	TRACE
	DEBUG
	INFO
	WARN
	ERROR

In a production environment, you will probably only see INFO and above published to the logExchange.

So, you could subscribe to all log data coming from HazCollectEx by binding your queue to the logExchange with the key:

	HazCollectEx.*

You could subscribe ERROR messages from all contexts by binding your queue to logExchange using the key:

	*.ERROR

Subscribe to INFO and above messages from all contexts by binding your queue to logExchange with multiple keys:

*.INFO
*.WARN
*.ERROR

Get all messages from all contexts with:

	*.*

You get the idea.

AMQP messages support arbitrary header data in the form of key-value paris and a message body or payload. All messages published to logExchange contain headers consistent with this example:

	{
		message=Using Java reflection API to modify options of Wss4jSecurityInterceptor to support sha256., 
		timestamp=1329497076408, 
		level=INFO, 
		threadName=main, 
		context=HazCollectEx, 
		loggerName=gov.noaa.nws.ipaws.IpawsService
	}

The message body is a string that is the fully formatted log message:

	2012-02-17 09:44:36,408 INFO [main] gov.noaa.nws.ipaws.IpawsService [IpawsService.java:37] Using Java reflection API to modify options of Wss4jSecurityInterceptor to support sha256.
	
This solution should provide enough information to effectively monitor issues within running Iris applications while providing the flexibility needed to integrate the data in to any environment-specfic monitoring system.