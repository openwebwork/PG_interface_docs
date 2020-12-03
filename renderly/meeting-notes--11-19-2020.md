# Metadata
## Problem specific
* What are the modes
  * public - goes to opl
  * private - stays local, only you can see
  * private variants:
    * Email is verified professor domain
    * Professor is validated by another professor
    * Professor is validated by an admin
    * User is picked by another user
* what are the content metadata
​
## Sequence
* What word are we going to use
* What does sequencing look like?
* About the problem in context of other problems
* Metadata
  * Type
    * bug fix
    * new problem
    * variant
​
# Unit testing
* Metadata with a required amount of "tests"
  * Example:
    * 5 unique random seeds must be selected
    * For each random seed it needs a list of answers
    * For each set of answers there should be a set of inputs that reach certain scores (at least 2, 100% and 0% correct)
​
# Linting
* Make sure no local macros are used
​
​
# Storage Ideas
How do we store this
## Option: closed (based on app permissions) CMS with open OPL 
* OPL flow stays exactly the same
* Users make edits to a problem in the database, possible collaboratively with multiple users
* When the user is ready they can ask for review or publish
* Publish would commit on behalf of the user and push it to the opl
* OPL review could now use data from `editor` to see upvotes etc.
* Permissions can be based n the mode from `problem specific`
​
# Notes
* Danny's wishlist
  * pg modified
    * see different parts of problems
    * Ex
      * Scaffolding
        * Some problems do this withuot scaffolding
    * idea here is to have a delimiter to separate parts
      * Could suppress parts
      * Could change value of parts
  * Would require modification of pg files and implementing the feature
​
​
# Action items
* Rederly
  * Think about backend
* Webwork
  * Think about metadata structure
  * Sequence document
