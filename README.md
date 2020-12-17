# Decision Recommendation

This toolkit showcases the Decision Recommendation APIs for IBM Business Automation Workflow on-premises installation with BA Machine Learning Server. Decision Recommendation provides recommended actions for workers who make decisions within a Workflow. An automated predictive machine learning model is created in the background using the process execution logs and business data in IBM Business Automation Insights, for any Workflow variable that you wish to get a recommendation on.

Toolkit provides:
1. Sample Workflow application with Decision Recommendation being used

2. Toolkit to integrate Decision Recommendation into your own workflows

3. Documentation on how to configure Decision Recommendation for your own workflows

## General Information ##

Decision recommendation can be used to recommend to the knowledge worker the value of a specifyied business object, by training on similar instances of the process and activity.


### How It Works ###

The recommendation is determined using a supervised classification model that predicts the target label. The features for the model are extracted through Process-Aware Feature Engineering methodology. It extracts various process-aware features using the process execution logs and business data to form the input feature vector for feature importance calculation and feature selection.

Intra-case features are extracted and loops/re-works are handled carefully for sequence representation. Because of the time-series nature of logs, features such as transitions, ordering, starters, categorical and numeric attributes are extracted in the form of one hot vector and normalized numbers respectively.

Significant features are then selected to train the models using automated feature selection.


### Business Value ###

Decision Recommmendation leverages the collective knowledge of the workforce on a task decision, thereby guiding workers in making more informed decisions. Such a pipeline would eliminate cognitive bias in workers while making decisions by standardising the way decisions are taken by different workers, providing fair decisions. It can save up to 25% of time and effort.


#### APIs

Two APIs are provided as part of decision recommendation, a train and a predict API.

	Train API: Give the process name, activity name and decision variable you wish to predict. Previous instance data is pulled from BAI and a machine learning model is created and stored in the DR server. Training must be done before predict to initialize the model in the server. 

	Predict API: Given the decisionVariable, the process name and activity name are computed. The current instance data is pulled from BAI and run through the machine learning model to generate a predicted recommendation for the decisionvariable. As well as a confidence expressed as a percentage and a natural language explanation of the recommendation, derived using the ELI5 Python library to determine the major contributing features learned by the model.



## Toolkit Information ##
The toolkit provides helpful Services, Business Objects and Views that can be used to ease integrating Decision Recommendation into your workflows.

The toolkit contains:

### Services: ###

#### Decision Recommendation Predict:
`Decision Recommendation Predict` calls the Decision Recommendation Predict API.

#### Decision Recommendation Train:
`Decision Recommendation Train` calls the Decision Recommendation Train API.

### Business Objects:
#### DecisionRecommendationResult
The Business Object that stores the results of the Decision Recommendation Predict API called from the [Decision Recommendation Predict](#decision-recommendation-predict) service. It contains:
  
  `DecisionVariable` (String): The variable for which Decision Recommendation is being asked to make a prediction
  
  `Prediction`(ANY): The value that decision recommendation recommends for `DecisionVariable`.
  
  `Recommendations` (String): A sentence explaining the 3 top reasons why a decision was taken.

#### DecisionRecommendationInput
This Business Object specifies the required input for the `Decision Recommendation Predict` Service. It contains:
  
  `decisionVariable` (String): The variable for which Decision Recommendation is being asked to make a prediction.
  
  `liveVariables` (NameValuePair) (List): A list containing information needed to enable the live data prediction feature (see the [Decision Recommendation with Live Data](#decision-recommendation-with-live-data-(optional)) section). It contains NamevaluePair Business Objects, each of which stores the tracking alias and value of a live variable.

#### DecisionRecommendationTrainInput
The Business Object that stores inputs to run the `Decision Recommendation Train` Service. It contains:
  
  `processName` (String): The name of the process where Decision Recommendation will be used.
  
  `decisionRecommendationModels` (ActivityDecisionVariablePair) (List): A list of Decision Recommendation models. Each model can be uniquely identified by activity name and decision variable; it is done so via the ActivityDecisionVariablePair Business Object.


### Views: ###

#### DecisionRecommendationView:
A view that displays the DecisionRecommendation prediction results. It shows the recommendation, the confidence percentage and the natural language explanation.

### Decision Recommendation with Live Data (Optional) ###
When calling Decision Recommendation on the current task, it uses changes last made from the prior completed activity. Optionally, if you wish to observe live how data from the current task affects the Decision Recommmendation outcome, you can enable the live data feature. With this feature enabled, users can get a recommendation that is up to date with live field changes in the coach.

For example, when carrying out the Negotiate Settlement task, if users wish to know how increasing the number of Waiting Days will affect the decision recommendation outcome on Allowed Duration of Disability (the decision variable), they can set up this live data feature to get an updated recommendation, simply by clicking the refresh button in `DecisionRecommendationView` after changing the number of Waiting Days.

#### Prerequisites ####
To use this feature, variables in the Workflow process, client-side human services and service flows should follow this naming convention:
1. Input, local and output variables in client-side human services, service flows that have a corresponding process variable should share the same name.
2. All process variable names and aliases should be identical. As there is a 30 character limit to variable tracking aliases, all variable names should be less than 30 characters such that the aliases match with the names.
3. All variable names within the Workflow process should be unique.

### Usage ####
Refer to Step 6 in the [Using Decision Recommendation in your Workflow](#using-decision-recommendation-in-your-workflow) section below.

## Installation and Configuration ##

1. Install and configure the Business Automation Machine Learing Server for Business Automation Workflow on-prem by following [these instructions](https://www.ibm.com/support/pages/node/6372710).

2. Ensure that Decision Recommendation is enabled by adding "dr" to the `SERVICES` variable in the `.env` config file

3. Download both the [sample app](Decision_Recommendation_Demo-1.0.twx) and the [toolkit](Decision_Recommendation_Toolkit-1.0.twx) twx files, then import to Workflow Center by following [these instructions](https://www.ibm.com/support/knowledgecenter/SS8JB4_19.x/com.ibm.wbpm.admin.doc/topics/managing_process_applications_E.html)

## Running the Sample 

1. Open the Disability Insurance Claim Process in ProcessPortal

2. Run the Disability Insurance Claim Process from Process Portal 

3. From there work through the Process and see on activities "Review Claim" and "Negotiate Claim" that Decision Recommendation is used to get a recommendation for the value of a boolean and numerical decision variable, respectively. The recommendation is visible as a green drop-down box.
 	
	Note: Predicitions will not initially work until sufficient tasks are completed and emitted into BAI, please complete the entire process a few times before checking for a Decision Recommendation Prediction. 

4. At the very end, you will see a system task, "Train Models" being run to train all models in the process.

## Using Decision Recommendation in your Workflow ##

Decision Recommendation can be used within your workflow using the Decision Recommendation Toolkit similarly to how it is used in the sample application. This is done by ensuring the process data is being emitted, the recommendation response is used and by scheduling a re-occuring training run.

1. Add the Decision Recommendation Toolkit as a dependency to your process application.

2. Take note of the following in regards to where decision recommendation will be added
   
   `processName`: name of process.
   
   `activityName`: the name of the activity.
   
   `decisionVariable`: the name of the variable you which for decision recommendation to predict. Decision variables can be any variable that is tracked by Business Automation Insights (BAI), such as an option the knowledge worker needs to select, any checkbox, any dropdown menu or any numerical input variable.

  You will need these in Step 4 and 6.


3. In the process that you want to add Decision Recommendation to
	1. In the Tracking section turn on autotracking 
  2. Turn on tracking on all process variables in the Variable section

4. Open to the activity you which to add the decision recommendation to in Process Designer
	1. Create a private variable of type `DecisionRecommendationInput` in the activity
	2. Check off the `hasDefault values` flag and enter in the `decisionVariable`
	3. Create a private variable of type `DecisionRecommendationResult`

5. Integrate the `decisionRecommendationView`
	1. If you already have a coach view for the activity open it, if not create a new coach view

	2. In the coach view add the `DecisionRecommendationView`
		a. Specify the following variables in the configuration section of the view:

		`DecisionResult`: The private variable of type DecisionRecommendationResult created in the previous steps
	        `autoRun`: Allow DecisionRecommendation to execute automatically once activity is loaded
				`Note: Moving too quickly through each activity may result in recommendations based on limited information, as time is required for events to load in BAI, if this occurs press the refresh button in the deicision recommendation view`

			`confidenceThreshold`: Percentage from 0-100% representing threshold of confidence model should reach before making a recommendation, recommendations with lower confidence than threshold will not be shown.

			`decision label`: Label to display along with the predicition, so knowledge workers know which field in the UI the predicition is for. 

	3. Connect the coach view to the Decision Recommendation Service
		1. Create a new service that calls the `Decision Recommendation Predict` Service
		2. In the Data Mapping section, map the private variable of type `DecisionRecommendationInput` and `DecisionRecommendationResult` previously created to their respective fields
		3. Drag and create a wire from the Coach to the DR service
		4. Set the binding of the wire End state binding to `click select` and select the refresh button in DecisionRecommendationViews
		5. Connect a `stay on page` intermediate Event to the DecisionRecommendation Service just created

 6. (Optional) Enable Decision Recommendation with live data
    1. Before the Decision Recommendation service, add a script to initialize and populate the `liveVariables` property in `DecisionRecommedationInput` with a chosen set of local variables. The scripts in the sample demonstrate how to include all local variables in the live data. This script will update `DecisionRecommendationInput` with live data before it is pass in to the `Decision Recommedation Predict` service.
	  2. The recommendation can now be updated by clicking the refresh button in `DecisionRecommendationView` when the activity is run.

7. Add model training service flow.
	Models need to be periodically refreshed with new data, to do this we will add a train service to train all our models.
	1. Define a service at the end of the process before the end node
	2. Let the service call the DecisionRecommendation Train Service
	3. Create a new BO of type `DecisionRecommendationTrainInput` and set the defualt values to: 
		- processName: the name of the current process
		- decisionRecommendationModels: For each decision recommendation implementation used in the process define the activityName and decisionVariable, as shown below, depending on the number of implementations in the process.
			```
			var autoObject = new tw.object.toolkit.DRT.DecisionRecommendationTrainInput();
			autoObject.processName = "Claim for Disability Insurance";
			autoObject.decisionRecommendationModels = new tw.object.listOf.toolkit.DRT.ActivityDecisionVariablePair();
			autoObject.decisionRecommendationModels[0] = new tw.object.toolkit.DRT.ActivityDecisionVariablePair();
			autoObject.decisionRecommendationModels[0].activityName = "First Activity Name";
			autoObject.decisionRecommendationModels[0].decisionVariable = "Decision variable used in first activity";
			autoObject.decisionRecommendationModels[1] = new tw.object.toolkit.DRT.ActivityDecisionVariablePair();
			autoObject.decisionRecommendationModels[1].activityName = "Second Activity Name";
			autoObject.decisionRecommendationModels[1].decisionVariable = "Decision variable used in second activity";
			autoObject
			```
	4. Map the new created BO to the decision recommendation train service


		

