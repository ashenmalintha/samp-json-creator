import groovy.json.*
import groovy.json.JsonOutput
import groovy.json.JsonSlurper
import groovy.transform.SourceURI
import java.nio.file.Path
import java.nio.file.Paths

// Todo: Get EXCEPT_INSTANCES from json file
def EXCEPT_INSTANCES = ['t1','nginx-router-dev-mig','nginx-router-dev1-mig','nginx-router-qa-mig','nginx-router-qa1-mig','ppay-nginx-router-dev-mig','ppay-nginx-router-qa-mig','cloud-sql-proxy-ce-npe-east','cloud-sql-proxy-npe-east','cloud-sql-proxy-npe-west','jenkins-agent-windows','nginx-router-dev-5rbt','nginx-router-dev-r325','nginx-router-dev1-5gk8','nginx-router-qa-7dvx','nginx-router-qa-xdm0','nginx-router-qa1-gttq','ppay-nginx-router-dev-wtbt','ppay-nginx-router-qa-69n4','ppay-nginx-router-qa-bt6z','tfm-loader-dev','tfm-loader-dev1','tfm-loader-qa','tfm-loader-qa1','tfm-backend-service-qa','tfm-billing-extract-qa','tfm-campaign-service-qa','tfm-consent-service-qa','tfm-ee-api-qa-8s1m','tfm-er-api-qa-tz19','tfm-es-web-qa-cn2p','tfm-mailer-qa','tfm-w2-account-manager-qa-4jfv','tfm-web-legacy-qa-4l8x']
def PPAY_JENKINS_PIPELINES = ['docker-test-abc*d-ss','ppay-ami-deploy-npe','ppay-ee-api-deploy-npe','ppay-email-notification-deploy-npe','ppay-er-api-deploy-npe','ppay-es-web-deploy-npe','ppay-iw4-deploy-npe','ppay-report-svc-deploy-npe']
def PROJECT = 'alert-rush-365303'
def BUILD_AGENT = 'npe-deploy-pod'
def json_filename = 'filename.json'
def selected_pipeline = "none"
def vm_name_crop_char_count = 7
def output_json_name= 'final_output_full.json'

pipeline {
    agent any
    stages {
        stage('json file creation') {
            steps {
                script {
                    
                    
                    sh '''  echo "{" > /tmp/final_output_full.json '''
                    sh(script :"gcloud config set project ${PROJECT}")
                    
                    
                    def instancesJsonString = sh(script :"gcloud compute instances list --format='json(name, zone)' --filter='status:(RUNNING)'", returnStdout: true)
                    println instancesJsonString
                    def instances = readJSON text: instancesJsonString
                    instances.each{value, key ->
                        
                        if(!(value.name in EXCEPT_INSTANCES)) {
                            
                            println "Test instance ${value.name}"
                            //json_filename = sh(script :"gcloud compute instances describe ${value.name} --zone=asia-southeast1-b --format='json(labels[version])' ", returnStdout: true)
                            json_filename = sh (
                            script: '''
                            gcloud compute instances describe '''+value.name+''' --zone='''+value.zone+'''  --format='json(labels[version])'
                            ''',
                            returnStdout: true
                            ).trim()
                            
                            //pipeline name selections function
                            selected_pipeline = pipeline_selector(value.name , PPAY_JENKINS_PIPELINES , vm_name_crop_char_count)
                            
                            //json builder fucntion
                            def json = json_builder(json_filename, selected_pipeline) 
                            
                            ////append into final output JSON file
                            File file = new File("/tmp/final_output_full.json")
                            file.append("\n")
                            file.append(json)
                            
                        } else {
                            println "INFO: Skipping MIG ${value.name}, EXCEPT_INSTANCES"
                        }
        
                        
                        
                        
                        
                        // json_filename = sh (
                        // script: '''
                        // gcloud compute instances describe '''+value.name+''' --zone=asia-southeast1-b  --format='json(labels[version])'
                        // ''',
                        // returnStdout: true
                        // ).trim()
                        
                        
                        
                        // json_filename = sh (
                        // script: '''gcloud compute instances describe docker-test --zone=asia-southeast1-b  --format="json(labels[version])" > filename.json
                        // cat filename.json
                        // ''',
                        // returnStdout: true
                        // ).trim()
                        
                        
                        //echo "Git committer email: ${json_filename}"
                        
                        //sh(script :"cat json_output")
                    
                    }

                    sh '''  echo "\n\n}" >> /tmp/final_output_full.json '''
                    sh '''  cat /tmp/final_output_full.json '''
	
		    sh '''  ls -al && pwd '''
		    git_push()
 
                }
            }
        }
    }
}

def git_push(){
    sh '''  rm -rf final_output_full.json '''
    sh '''  cp /tmp/final_output_full.json FINAL_OUTOUT.json '''
    sh '''  git add * '''
    sh '''  git commit -m "push JSON file" '''
    
	sh '''  ls -al && pwd '''
	sh '''  cat FINAL_OUTOUT.json '''
    sh '''  git push origin HEAD:main '''
    sh '''  git push '''
}

def String pipeline_selector(String vm_name, def PPAY_JENKINS_PIPELINES, int vm_name_crop_char_count){
    //def PPAY_JENKINS_PIPELINES = ['ppay-ami-deploy-npe','ppay-ee-api-deploy-npe','ppay-email-notification-deploy-npe','ppay-er-api-deploy-npe','ppay-es-web-deploy-npe','ppay-iw4-deploy-npe','ppay-report-svc-deploy-npe']
	String repopulated_vm_name = vm_name.substring(0, Math.min(vm_name.length(), vm_name_crop_char_count));
	println("testing for char selection fucntion")
	println(repopulated_vm_name)
	
	def selected_pipeline = "none"	

    for(pipeline in PPAY_JENKINS_PIPELINES) {
        //  println(pipeline);
        //  if (vm_name in pipeline){
        //      println("match found")
        //  }
         
        if (pipeline.contains(repopulated_vm_name)){
             selected_pipeline = pipeline
            //  println(" found matching part of pipeline name")
            //  println(pipeline)
        }
         

    }
    return  selected_pipeline
    
}


def String json_builder(String json_filename , String selected_pipeline) {
    // Read Input JSON File and extract the version
    // if (args.size() < 1) {
    //     println("Missing filename")
    //     System.exit(1)
    // }
    //filename = args[0]
    //filename = json_filename

//     Path scriptLocation = Paths.get(json_filename)
//     println(scriptLocation)
//     println(scriptLocation.getParent())  
// def current_script = getClass().protectionDomain.codeSource.location.path
// println(current_script)
    
    // def jsonSlurper01 = new JsonSlurper()
    // input_data = jsonSlurper01.parse(new File(scriptLocation))

    // def input_json_str = JsonOutput.toJson(input_data)
    //def input_json_str_pretty_format = JsonOutput.prettyPrint(json_filename)
    def json = new groovy.json.JsonSlurper().parseText(json_filename)
    version = json.labels["version"]

    version = version.replaceAll("\\-",".")

    ////Create new data set for build JSON block
    def data = [

        "job": "$selected_pipeline",
        params: [ "branch": "develop",
        "version": "$version"]
    ]

    //Genarate new JSON Block
    def json_str = JsonOutput.toJson(data)
    def json_output = JsonOutput.prettyPrint(json_str)
    println(json_output)
    
    return json_output
}

