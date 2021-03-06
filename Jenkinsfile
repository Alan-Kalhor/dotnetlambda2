//def bucket = 'deployment-packages-test1'

node {

	
	def DOTNET_PATH = '/home/ec2-user/dotnet'
	def FUNCTION_NAME = 'dotnetlabmda2'
	def APP_MAIN_FOLDER = 'dotnetlabmda2'
	def S3_BUCKET = 's3.alankalhor.dotnetlabmda2'
	def REGION = 'ap-southeast-2'
	def PROD_ALIAS = 'production'
	def STAGING_ALIAS = 'staging'

	
	def lambdaVersion = ''
	
    stage('Checkout'){
        checkout scm
    }
	
	stage('Build') {
		sh "sudo $DOTNET_PATH/dotnet clean"	
		sh "sudo $DOTNET_PATH/dotnet build --configuration Release"
	}
	
	stage('Test') {
		sh "sudo $DOTNET_PATH/dotnet test"
	}
	

	stage('Deploy') {
		env.DOTNET_ROOT = "/home/ec2-user/dotnet"
		env.PATH = "$PATH:/home/ec2-user/dotnet"

		sh "$DOTNET_PATH/dotnet-lambda list-functions"
		
		//sh "aws --region ${REGION} cloudformation create-stack --stack-name ${FUNCTION_NAME}"
		sh "echo 'about to create s3'"		
		sh "aws s3api create-bucket --bucket ${S3_BUCKET} --create-bucket-configuration  LocationConstraint=${REGION}"
		
		sh "echo 'about to deploy lambda'"		
		dir("${APP_MAIN_FOLDER}") {
			//sh "$DOTNET_PATH/dotnet-lambda deploy-function DotNetCoreWithTest1 --function-role JenkinsBuildRole"
			sh "$DOTNET_PATH/dotnet-lambda deploy-function --function-runtime dotnetcore2.1 --function-name ${FUNCTION_NAME}  --function-memory-size 256 --function-timeout 30 --function-role mydotnetroll --function-handler ${FUNCTION_NAME}::${FUNCTION_NAME}.LambdaEntryPoint::FunctionHandlerAsync --disable-interactive true"
			
			sh "echo 'after deploy-function'"						
			
			sh "$DOTNET_PATH/dotnet-lambda deploy-serverless ${FUNCTION_NAME}"

		}
	}

	
	if (env.BRANCH_NAME == 'master') {
		stage('Publish') {
		
			lambdaVersion = sh(
				script: "aws lambda publish-version --function-name ${FUNCTION_NAME} --region ${REGION} | jq -r '.Version'",
				returnStdout: true
			)
			sh "echo $lambdaVersion"						
		}


		stage('Deploy to Staging?') {
			milestone 2
			input "Do you want to deploy to Staging?"
		}

		stage('Deploy to Staging') {
			def check_staging_alias = sh(
				script: "aws lambda list-aliases --function-name ${FUNCTION_NAME} --region ${REGION} | jq -r '.Aliases[] | select(.Name == \"${STAGING_ALIAS}\") | 1'",
				returnStdout: true
			)
			
			if (!check_staging_alias.trim().equals("1")) {
				sh "aws lambda create-alias --function-name ${FUNCTION_NAME} --name ${STAGING_ALIAS} --region ${REGION} --function-version ${lambdaVersion}"
			}
			else {
				sh "aws lambda update-alias --function-name ${FUNCTION_NAME} --name ${STAGING_ALIAS} --region ${REGION} --function-version ${lambdaVersion}"			
			}
		}
		
		stage('Deploy to Production?') {
			milestone 3
			input "Do you want to deploy to Production?"
		}		
		
		stage('Deploy to Production') {
			def check_prod_alias = sh(
				script: "aws lambda list-aliases --function-name ${FUNCTION_NAME} --region ${REGION} | jq -r '.Aliases[] | select(.Name == \"${PROD_ALIAS}\") | 1'",
				returnStdout: true
			)
		
			if (!check_prod_alias.trim().equals("1")) {
				sh "aws lambda create-alias --function-name ${FUNCTION_NAME} --name ${PROD_ALIAS} --region ${REGION} --function-version ${lambdaVersion}"
			}
			else {
				sh "aws lambda update-alias --function-name ${FUNCTION_NAME} --name ${PROD_ALIAS} --region ${REGION} --function-version ${lambdaVersion}"
			}		
		}
	}	
	
	
}
