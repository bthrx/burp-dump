PortSwigger’s BurpSuite is the de facto tool for web, API, and mobile application assessments.  Over the course of many engagements, the utility of being able to filter and extract data from BurpSuite to the file system became evident.  Why? A few reasons…

* BurpSuite provides functionality to log to text files.  However, it is not currently possible to control which requests/responses are logged.  Limiting logging to application hosts that are designated as in scope is helpful to security analysts.

* BurpSuite provides search functionality as well.  However, there are certain use cases, such as searching on the results of a search, where a combination of *nix CLI tools, such as grep, sed, awk, etc, can be more useful.  

* BurpSuite provides a comment functionality that can be used to annotate requests / responses, a feature useful to analysts to provide additional information and context to logged data.  However, Burp comment data is not reflected in BurpSuite’s logging output.

* Often, analysts perform a basic functionality walkthrough an application to understand the application and observe traffic generated during typical use.  If a security consultant forgets to enable logging beforehand, the process must be repeated after enabling logging.

* Also, other tools are able to parse BurpSuite’s text log format in order to perform tasks (for example, sqlmap’s -l option).

Dump was written to address these issues.  It is a simple BurpSuite extension written in JRuby that exports HTTP(S) requests and responses for specified hosts in two different log formats.

**Log Formats**

* Burp - This format is nearly identical to the output created by the option available under Project Options, Misc, Logging.  The result is a single text file containing all HTTP(S) requests and responses.  The advantage of using dump over the stock functionality is that a host filter may be specified and the dump plugin must be run after the requests/response are recorded (as opposed to enabling logging beforehand).

* WebScarab - This format is nearly identical to output generated by the deprecated OWASP WebScarab project.  HTTP(S) request and response pairs are assigned an ordered integer and named according to its nature (e.g. 1-request.txt and 1-response.txt).  Again, a host filter may be specified to ensure only traffic associated with particular hosts is in the log files.

*Note*: Due to a limitation of accessing certain types of metadata within the Burp Extender API, please note that timestamps and the IP address entries in metadata header of the Burp log file format are faked, as highlighted in this screenshot:

![Faked Burp log file metadata](/content/images/2017/07/DumpPluginFakedData_SublimeText_Highlights.png)

However, the URI portion is valid and reliable. Also, the original HTTP(S) request and response are unadulterated, with exception of the Burp Comment Header inserted by the dump plugin (unless disabled). 


**Hostname Filtering**

The host filter is used to specify which hosts should be included the log output.  For example,

* A wild card character (*) includes all requests/responses from Burp’s proxy history.

* A single hostname (e.g. blog.stratumsecurity.com) will filter out all requests and responses not destined for blog.stratumsecurity.com

* A list of hostnames separated by a comma (e.g. blog.stratumsecurity.com, www.stratumsecurity.com)

*Note*: A partial match of the specified hostname will cause a match to occur and therefore will be included in log output.  For example, if stratumsecurity.com were specified, then requests bound for blog.stratumsecurity.com and www.stratumsecurity.com will be present in the log output.

**Burp Comments / Context**

In order to preserve comments, the dump plugin is capable of inserting a custom HTTP header containing the Burp comments associated with HTTP(S) request/response pair.  In order to accomplish this, dump inserts the X-Burp-Comment: header (by default, this is customizable)  and inserts the current comment as data into the header.

Don’t want to insert Burp comments into log files?  No problem – simply remove all text from the Dump's Burp Comment Header text field before dumping.

Additionally, a simple construct called Context is recognized by the dump plugin.  The purpose of Contexts is to allow multiple Burp comments to be inserted into one instance of the custom HTTP header.  The dump plugin recognizes any Burp comment containing the Context Separator value (!!! by default) as a context and all subsequent log entries will contain the Burp Comment identified as a Context, until updated by a different Burp comment also containing the Context Separator value.

One use for Contexts is to clearly identify which user is logged in when important transactions occur.  For example,

* The analyst browses to the application, enables Intercept, and logs into the application.

* The HTTP(S) request containing a login request for John, a user with basic application
privileges, is intercepted by Burp.

* Within Burp, the analyst annotates the request with a comment such as “!!! John Login - Basic User” and forwards the request or disables interception.  This dump plugin notes this specific comment as the first Context.

* The analyst continues the application walkthrough and intercepts requests for important transactions within the application.  All intercepted requests are given meaningful comments, such as “Change Password” or “Transfer Funds”.

* The analyst logs out of the application.

* Within the same Burp session, the process is repeated for all users (e.g. Mary), each with a different application role.

* After completing the walkthrough, the analyst uses Dump to create the log file(s).
Observing the different X-Burp-Comment Headers in the log file(s) will reveal:

X-Burp-Comment: Change Password  |  !!! John Login - Basic User
X-Burp-Comment: Transfer Funds  |  !!! John Login - Basic User
X-Burp-Comment: Change Password  |  !!! Mary Login - Admin User
X-Burp-Comment: Create User Account  |  !!! Mary Login - Admin User
X-Burp-Comment: Change User Account Email  |  !!! Mary Login - Admin User

**SEE IT IN ACTION**

A few videos that demonstrate how to use Dump can be found here:

https://www.youtube.com/watch?v=RzfH2YdCe7g&list=PLZ78xiCeE7VyHnxP5ESBYyeW2DmVghKME

**CODE**

Available at GitHub - https://github.com/crashgrindrips/burp-dump
