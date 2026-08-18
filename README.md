
javax.net.ssl.SSLProtocolException: Illegal server name, type=host_name(0), name=.aol.co.in, value={2E7362692E636F2E696E}
Caused by: java.lang.IllegalArgumentException: The encoded server name value is invalid
Caused by: java.lang.IllegalArgumentException: Empty label is not a legal name

i have a ssl certificate issued to *.aol.co.in what proper server names will this accept as iam getting javax.net.ssl.SSLProtocolException: Illegal server name, type=host_name(0), name=.aol.co.in, value={2E7362692E636F2E696E} Caused by: java.lang.IllegalArgumentException: The encoded server name value is invalid Caused by: java.lang.IllegalArgumentException: Empty label is not a legal name
