# Renderer Refactoring Project (RRP)

###  references to GitHub line numbers are for the master version (which is less likely to change)

###  The core of the renderer

1. `$pg = WeBWorK::PG->new(INPUT)`

   The actual work is done in `WeBWorK::PG::Local` which is called from `WeBWorK::PG->new()` The extra indirection allows the possibility of transparently calling different renderers -- e.g. a round-robin using `PG::Remote` instead of `Local` inorder to distribute load.  

   1. input snippet: https://github.com/openwebwork/webwork2/blob/master/lib/WeBWorK/PG/Local.pm#L82
   2. output snippet: https://github.com/openwebwork/webwork2/blob/master/lib/WeBWorK/PG/Local.pm#L468
   3. POD documentation: https://demo.webwork.rochester.edu/wwdocs
   4. The naming and organization of the input interface could be improved. There are subhashes of the input hash which could be reorganized making it clearer what is required and what had default settings.  The output interface is not as bad but could also be reorganized with less redundancy. 
   5. The environment input hash also needs  reorganization -- it has been added to and some old items were maintained for backward compatibility and can probably be safely removed at this point. 

2. `https://github.com/openwebwork/webwork2/wiki/PG-problem-rendering-execution-path` gives an overview of the rendering of the problem from the .pg file to produce either an HTML or a PDF (or potentially some other representation) of the question. 

3. It includes checking the student's answers.  In order to check the student answers the entire problem is rendered to obtain the correct answer evaluators and then those are applied to the student's answers to obtain a score for each problem.

4. While the rendering process itself inside `PG::Local` can use refactoring work that is not the focus of the current process.  All of the use cases below run through the `$pg = WeBWorK::PG->new(INPUT)` command.  However they ways that they gather the inputs (and to a lesser degree what they do with the outputs stored in the $pg object) are similar but inconsistent and would benefit from refactoring and a reorganization.

5. In particular choosing better and consistent names for the inputs and outputs will make the next round of upgrades easier. 

### Use cases of the renderer in webwork2

1. ``lib/ContentGenerator/Problem.pm`, the oldest version of using WeBWorK::PG

   ```
   	my $pg = WeBWorK::PG->new(
   		$ce,
   		$effectiveUser,
   		$key,
   		$set,
   		$problem,
   		$set->psvn, # psvn (problem set version number)is historical, not used much and is effectively a setID number.
   		$formFields,
   		{ # translation options
   			displayMode     => $displayMode,
   			showHints       => $will{showHints},
   			showResourceInfo => $will{showResourceInfo},
   			showSolutions   => $will{showSolutions},
   			refreshMath2img => $will{showHints} || $will{showSolutions},
   			processAnswers  => 1,
   			permissionLevel => $db->getPermissionLevel($userName)->permission,
   			effectivePermissionLevel => $db->getPermissionLevel($effectiveUserName)->permission,
   		},
   	);
   ```
   ```
   
   
   ```

2. `lib/ContentGenerator/GatewayQuiz.pm`

   1. the renderer is called at https://github.com/openwebwork/webwork2/blob/master/lib/WeBWorK/ContentGenerator/GatewayQuiz.pm#L2349. It is inside the subroutine `getProblemHTML`. The for loop which renders each of the problems is at https://github.com/openwebwork/webwork2/blob/master/lib/WeBWorK/ContentGenerator/GatewayQuiz.pm#L1257.
   2. The GatewayQuiz file is particularly monolithic.
   3. In order to store GatewayQuiz data in the current database hacks were used to store the version number of the quiz in the name. (Students are allowed to take the same quiz more than once and each attempt at the quiz is a version.)  Adding the proper fields to the database would improve the logic and readability of this file. 

3. `lib/WebworkWebservice/Renderer.pm`

   1. This is the oldest version of the webservice whose front end path is 
   
      `<Location /mod_xmlrpc>   -> WebworkXMLRPC -> WebworkWebservice::RenderProblem.pm`
   
   2. The file WebworkWebservice.pm should be reorganized, it contains a lot of commented out legacy code and experiments. The sub package WebworkXMLRPC defined in the file should be replaced by package `WebworkWebservice` named at the top of the file, and  other modifications made as needed. The name `WebworkXMLRPC` should be removed and the package `WebworkWebservice` will serve as a dispatcher for web services.
   
   3. Other web services are also dispatched from the `WebworkWebservice` file but for this refactoring we need only be concerned with `WebworkWebserice::RenderProblem.pm`
   
   4. There is also a `ContentGenerator::ProblemRenderer` which renders a problem with a minimum amount of UI garbage.  It uses `Tasks::renderProblem` internally. 

4. `lib/Utils/Tasks.pm (renderProblems)`
   1. This version rewritten  by John Jones and will render and return more than one problem.  It is short and cleanly written.  There is only one major task in this file -- `renderProblems` -- but it also contains subroutines to create fake data which supplies defaults to `renderProblems()`These auxiliary subroutines `fake_user` and `fake_set` are used in many places -- even in places where Tasks/renderProblems is not used.  The question marks indicate places where I think the use statement is including `renderProblems` incorrectly.
   2. `Tasks::renderProblems` is used to display problems from the `PGProblemEditor`, in the `GetLibrarySetProblems`, `GetTargetSetProblems` `ProblemSetDetail`, `ProblemSetDetail2` 
   3. Is it possible to replace `WebworkWebserver/RenderProblem.pm` and the call in `Problem.pm`with a call to this Tasks version?

5. `lib/ContentGenerator/renderViaXMLRPC.pm`

   1. accepts an HTTP call, translates the arguments and passes the call to `WebworkClient` which then calls `WebworkWebservice` via HTTP to have `ProblemRenderer` render the problem and return it back up the chain. .

6. `lib/ContentGenerator/instructorXMLHandler.pm`

   1. Does much the same thing as `renderViaXMLRPC` but accepts internal calls  generated by javascript files associated with the library browser and homeworksetEditor (`setmaker.js` and `tagwidget.js`and `problemsetdetail2.js`) and passes them via `WebworkClient`, `HTTP`, and `WebworkWebservice` to `ProblemRender` and to some `WebworkWebservice/LibraryFunctions` .  It seems to have a lot of code which is not being used. 
   2. I think the essential difference between `renderViaXMLRPC.pm` and `instructorXMLHANDLER.pm` is that the former has `xml` transport for input and `xml` output to the web service.  The latter has `JSON` transport for input and `xml` output to the web service.  There are details I don't understand about the calls from the .js files.  We should ask John Jones if he has time to help out (he wrote the LibraryBrowser)

7. `lib/WebworkClient.pm`

   1. This is a complicated file, slowly being dissected into its parts.  

   2. The main remaining function of this file is to provide xmlrpc_call() which calls the webworkwebservice.

   3. It is used by `renderViaXMLRPC.pm`, `instructorXMLHandler.pm` and also by `clients/sendXMLRPC.pl` (the command line editor interface for PG) to construct a client which sends an xmlrpc request to the WebworkWebservice. 

      `FormatRenderedProblem.pm` The content of this file was split off from WebworkClient.pm.  It is responsible for formatting the output of xmlrpc_call() when the output is not directly passed on to some other subroutine. It also formats the output created by the `standalonePGproblemEditor` which is why it was split off. It uses format templates in `WebworkClient` folder (a bad name). The FormatRenderedProblem function has overlaps with Webwork::Template and the templates in webwork2/htdocs/themes but that is another refactoring project -- not this one.
      
   4. !!!! I have some more work to do in untangling the code in this file. It was the first file written in the webwork webservice sequence and has been hacked on considerably since then by several people.  The main remaining purpose is xmlrpcCall() which initiates an xmlrpc request through SOAP::Lite cpan module.  From what I've read SOAP::Lite is largely deprecated in favor of newer modules supporting xml transport. It's also possible that we could use JSON transport instead.   

   5. The currently active subroutines seem to be

      1. xmlrpcCall() - which sends the render command to the webworkWebservice
      2. new which creates the client object
      3. environment() which builds a default PG environment for a problem. (similar to code in PG.pm)
      4. default_inputs() which supplies defaults such as the CPAN modules being made available to the safe compartment.  (similar to code PG::Local.pm)
      5. Utilities:  a collection of accessors to retrieve values in the xmlrpcClient object.

   6. viewing sendXMLRPC.pl gives some idea of how the functions in WebworkClient are used. 



1. `lib/ContentGenerator/RenderProblem.pm` (possibly cruft at this point??)   It uses `Tasks::renderProblem` internally to render a problem without including much UI cruft.


### Other use cases of the renderer

1. `OpaqueServer.pm`
   1. Uses SOAP transport instead of XMLRPC.  Replicates the WebworkWebservice/RenderProblem.pm function.  It's main use is to render individual problems in Moodle quizzes. The calling client is a moodle module called Opaque client written originally by Tim Hunt which was designed to interface with arbitrary problem rendering engines.
2. `clients/sendXMLRPC.pl`
   1. This demonstrates the cleanest use of using the webworkWebservice rendering chain for obtaining HTML representations, tex, or pdf representations of the pg problem. One can also obtain much of the internal structure built while rendering student answers. The POD documentation is quite good, for a change. 
3. `standaloneProblemEditor.pm`
   1. This is a clone of sendXMLRPC (see github.com/mgage ) but instead of calling the renderer through the webworkWebservice it accesses the Utilities/Task.pm renderer directly. It is not quite as polished as sendXMLRPC.pl but the idea is to make it behave functionally as an exact clone of sendXMLRPC.pl when called from a command line. It's advantage is that it does not require a webserver such as Apache. 
4. `Mojalicious (Rederly Server)`
   1. This was built using standaloneProblemEditor.pm as a model but doesn't include all of it's extra functions (providing pdf output for example). Alex Jordan is the author.

### Uses of Utils::Tasks::renderProblem
* imported to and called from `GetTargetSetProblems` -- only used by SetMaker2 which is now deprecated??
* imported to and called from `GetLibrarySetProblems` -- only used by SetMaker2 which is now deprecated??
* imported to and called from Compare.pm  -- not currently being used
* imported to and called from `ProblemSetDetail`
* imported to and called from `ProblemSetDetail2` ? is this different or just newer version of ProblemSetDetail???
* imported to and called from `SetMaker`
* imported to and called from `SetMakernojs`
* imported to `Achievement  Editor`
* imported to SetMaker2 -- David Gage's version -- now deprecated
* imported to SetMaker3 -- ??
* imported to `PGproblemEditor`


### Unused modules (part of LibraryBrowser2 (setmaker2) David Gage's project which was never completed for prime time)

* it used javascript to display the library problems visually in a gallery from which you could choose problems for your course via drag and drop.
* `setmaker2.pm`
* `GetTargetSetProblems.pm` -- only used by SetMaker2 which is now deprecated??
* `GetLibraryProblems.pm` -- only used by SetMaker2 which is now deprecated??
* `teacher.js` -- only used by SetMaker2 which is now deprecated??


### SOAP::Lite and xmlrpc

* My impression is that the SOAP Lite modules (which includes xmlrpc) are obsolete and can be replaced with more modern perl modules for xml transport.  If we do nothing more we should upgrade these xmlrpc transport modules.
* We should also look into using JSON transport instead of or in addition to XML transport. 

# distribution list

* Tom (rederly)
* Joe (rederly)
* Peter Staab
* Glenn Rice
* Drew Parker
* Alex Jordon
* Mike Gage
* Nathan Wallach
* Paul Pearson
* John Jones
* Gavin Larose
* Danny Glina