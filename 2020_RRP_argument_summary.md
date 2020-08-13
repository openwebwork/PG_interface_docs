### iframe call to render problem

Key  | value | consumed in |
-----|-------| -------------
src	  |	... webwork2/html2xml?	| renderViaXMLRPC.pm |
answersSubmitted | 0 						   |          |
sourceFilePath   |Library/.../ filename.pg | PG->new  |
problemSeed      |43256/ | PG_envir (envir) 
courseID			| daemon_course | WebworkWebservice   |
userID           | daemon        | WebworkWebservice   |
course_password  | daemon        | WebworkWebservice   |
showSummary      |               |FormatRenderedProblem|
displayMode      | MathJax       | PG_envir   |
problemUUID      |  3243         | PG_envir   |
language         | en            | PG_envir   |
outputformat     | libretexts    | FormatRenderedProblem|
webworkJWT       |aaa.bbb.ccc    | encoded JWT immutable data

### this GET request is received and preprocessed by renderViaXMLRPC 
* The GET arguments are in `%inputs_hash
* Adjustments are made to key names to accomodate LTI calls (e.g. from Canvas)
	* for example `custom_answerssubmitted` => `answersSubmitted`
* webworkClient is created with pass through values from input_hash
	
	| key    | obtained from   | value| consumed by |    
	|--------|-----------------|-----|------------|
	|site_url| SITE_URL        |url of webworkwebservice site| xmlrpcCall()|
	|site_password | | password for using webworkwebservice  site|
	|form_action_url| FORM_ACTION | inserted into problem form for checking |
	|userID | input_hash 			| passthrough|
	|courseID | input_hash 			| passthrough|
	|course_password | input_hash | passthrough|
	|session_key| input_hash  | returned by webworkwebservice, can be used instead of password|
	|outputformat| input_hash | passthrough|
	|webworkJWT  | input_hash | passthrough|
	|input_hash  | \%input_hash| includes many pg_envir values|
	
* This hash is posted to the `WebworkXMLRPC::renderProblem` in Webworkwebservice.pm by `xmlrpcCall()`	
* `$xmlrpc_client->xmlrpcCall(renderProblem, $xmlrpc_client->{input_hash}) )`
* notice that values in input_hash are being carried in parallel by as much as 3 channels: `xmlrpc_client`,`xmlrpc_client->input_hash`,`input_hash`  (this needs pruning)
* notice also that we don't need both webworkwebservice and webworkXMLRPC

###  `WebworkXMLRPC::renderProblem`
* renderProblem first calls `initiate_session` which

	* creates WebworkXMLRPC object using values from the hash reference $rh_input.

	* |key | from |
|----------|------|
|		courseName|`$rh_input ->{courseID}` |
|     `user_id`	|`$rh_input->{userID}`    |
| password       |`$rh_input->{course_password}`|
|`session_key`     |  `$rh_input ->{session_key}`|

	* authenticates the user_id, courseName, password using the webwork2 authorization routines (slightly modified with a mixin)
* The name changes are probably necessary to match values used in checking the authorization using the webwork legacy machinery
* The method for calling the webwork2 code to authenticate is squirrly because it needs to interface with the webwork2 authentication methods.  It needs to be checked -- how is it being done?  is it being done? originally it was done by renaming some of the subroutines. (mixins)  Is that still being done?
 
* the initiate_session call returns an object whose `do` method performs the rendering 

``` 
renderProblem {
...
my $self = $class->initiate_session($in);
return $self->do( WebworkWebservice::RenderProblem::renderProblem($self,$in) );
$self is a webworkXMLRPC object
```

* The output of the `do` call is passed back as the return value for the `xmlrpcCall()`
	1.  `$self` should be replaced with something like `$webworkXMLRPCobject`
	2. replace the `do` call with something more function oriented?
	
### renderProblem process
* input_hash to Webworkwebservice::RenderProblem::renderProblem()
* call to PG::defineEnvironment
* create a PG object (PG->new)
	* input hash to PG->new
	* PG object automatically processes the inputs and creates an output hash
	* output hash from PG->new

### WebworkClient.pm
	* we're back from the xmlrpcCall()
	* feed the output of that to FormatRenderedProblem
	* feed the output of FormatRenderedProblem back to renderViaXMLRPC
	* feed the return value of that to the Browser
### FormatRenderedProblem
* uses a format template in WebworkClient directory
