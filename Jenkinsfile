# Jenkinsfile

// --------------------------------------------------
// Load Jenkins Shared Library
// --------------------------------------------------

@Library('jenkins-shared-library') _

// --------------------------------------------------
// Application Configuration
// --------------------------------------------------

def configMap = [

    // Project name
    project : "roboshop",

    // Component name
    component: "frontend"
]

// --------------------------------------------------
// Branch Validation
// --------------------------------------------------

// Execute pipeline only for non-main branches

if( ! env.BRANCH_NAME.equalsIgnoreCase('main') ){

    // Call reusable shared library function
    nodeJSEKSPipeline(configMap)

}
else{

    echo "Please proceed with PROD process"
}