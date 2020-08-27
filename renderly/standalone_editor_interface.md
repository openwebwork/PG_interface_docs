# Renderly Standalone Editor API
Can be interfaced through `/render-api`

# Parameters
| Key | Type | Default Value | Required | Description | Notes |
| --- | ---- | ------------- | -------- | ----------- | ----- |
| sourceFilePath | string | null | true if problemSource is null | The path to the file to be rendered | Can begin with Library or Contrib, in which case the renderer will automatically adjust the path from the webwork-open-problem-library root |
| problemSource | string | null | true if sourceFilePath is null | The source of a problem to render | base64 encoded raw pg content - can be used instead of sourceFilePath |
| problemSeed | number | NA | true | The seed to determine the randomization of a problem | |
| formURL | string | localhost:3000 | false | the URL for form submission | |
| baseURL | string | localhost:3000 | false | the URL for relative paths | |
| outputFormat | string (enum) | static | false | Determines how the problem should render, see below descriptions below | |
| language | string | en | false | Language to render the problem in (if supported) | |
| showHints | number (boolean) | 1 | false | Whether or not to show hints (restrictions apply) | Hint logic has some hooks into the problem source, if permission level is high enough or `n` number of attempts has been reached they will render if this is true (1), however if false (0) you will never see the hints)
| showSolutions | number (boolean) | 0 | false | Whether or not to show the solutions | |
| permissionLevel | number | 0 | false | Aids in the conrol of show hints, also controls display of scaffold problems (possibly more) | See the levels we use below |
| problemNumber | number | null | false | We don't use this | |
| numCorrect | number | 0 | false | The number of correct attempts on a problem | |
| numIncorrect | number | 1000 | false | the number of incorrect attempts on this problem | Relevant for triggering hints that are not immediately available |
| processAnswers | number (boolean) | 1 | false | Determines whether or not answer json is populated, and whether or not problem_result and problem_state are non-empty | |

# Output Format
| Key | Description |
| ----- | ----- |
| static | zero buttons, locked form fields (read-only) |
| single | one submit button (intended for graded content) |
| simple | submit button + show answers button |

# Permission level
| Key | Value |
| --- | ----- |
| student | 0 |
| prof | 10 | 
| admin | 20 |

# Permission logic summary for hints and solutions
* If professor or above (permission value >= 10) you will always get hints and solutions
* If student you will get solutions if and only if we set the showSolutions flag to true
* If student you will get hints after you have exceeded the attempt threshold (if the flag is set to true)
