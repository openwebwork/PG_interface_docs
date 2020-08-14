# Environmntal Variables for the PG Render

This is a description of the environmental variables needed for the renderer.

It appears the main variables/fields needed are those from the `ansEvalDefaults`, which taken directly from `webwork2/conf/defaults.config`: 

```
$pg{ansEvalDefaults} = {
	functAbsTolDefault            => .001,
	functLLimitDefault            => .0000001,
	functMaxConstantOfIntegration => 1E8,
	functNumOfPoints              => 3,
	functRelPercentTolDefault     => .1,
	functULimitDefault            => .9999999,
	functVarDefault               => "x",
	functZeroLevelDefault         => 1E-14,
	functZeroLevelTolDefault      => 1E-12,
	numAbsTolDefault              => .001,
	numFormatDefault              => "",
	numRelPercentTolDefault       => .1,
	numZeroLevelDefault           => 1E-14,
	numZeroLevelTolDefault        => 1E-12,
	useBaseTenLog                 => 0,
	defaultDisplayMatrixStyle     => "[s]",  # left delimiter, middle line delimiters, right delimiter
	enableReducedScoring          => 0,
	reducedScoringPeriod          => 0,	# Default length of Reduced Scoring Period in minutes
	reducedScoringValue	     => .75,	# Percent of score students recieve in Reduced Scoring Period
	timeAssignDue				  => "11:59pm",
	assignOpenPriorToDue	      => 14400, # a number of minutes (default is 10 days)
	answersOpenAfterDueDate       => 2880 # number of minutes (default is 2 days)
};
```
