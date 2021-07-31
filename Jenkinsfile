import groovy.json.JsonSlurper
import java.time.*
import java.time.format.DateTimeFormatter

def summary = []
def files = []

pipeline {
    /** The agent should be changed based on the Jenkins instance. **/
    agent { label 'master' }
	environment {
		GIT_URL = "https://github.com/kunalkumar229/jenkins-pipeline-demo" // Git repository should contain build.json file.
		BUILD_FILE_PATH = "build.json"
		ZIP_FILE_NAME = "builds.zip"
		IS_TODAY_HOLIDAY = false
	}
	parameters {
		booleanParam 'Unit_Test'
		booleanParam 'QA'
		booleanParam 'Static_Check'
		string defaultValue: 'success@test.com', name: 'Success_Email'
		string defaultValue: 'failure@test.com', name: 'Failure_Email'
	}
	stages {
		stage('Git Pull') {
		    steps {
			    script{
					cleanWs()
					git GIT_URL
			    }
		    }
		}
		stage('Is the run required?') {
			options {
				retry(3)
			}
		    steps {
				script {
					IS_TODAY_HOLIDAY = getHoliday()
				}
			}
		}
		stage('Build') {
			when {
				expression { IS_TODAY_HOLIDAY == false } 
			}
            steps {
				script {
					files = build(BUILD_FILE_PATH,ZIP_FILE_NAME)
				}
           	}
        }
		stage('Parallel Stages') {
			when {
				expression { IS_TODAY_HOLIDAY == false }
			}
			parallel {
				stage('Quality'){
					stages{
						stage('Static check') {
							when {
								environment name: 'Static_Check', value: 'true'
							}
							steps {
								script{
									summary.add([name: "Static check", files: files])
									unzip dir: 'Static_check', glob: '', zipFile: ZIP_FILE_NAME
								}
							}
						}
						stage('QA') {
							when {
								environment name: 'QA', value: 'true'
							}
							steps {
								script{
									summary.add([name: "QA", files: files])
									unzip dir: 'QA', glob: '', zipFile: ZIP_FILE_NAME
								}
								
							}
						}
					}
				}
				stage('Unit test'){
					when {
						environment name: 'Unit_Test', value: 'true'
					}
					steps {
						script{
							summary.add([name: "Unit test", files: files])
							unzip dir: 'Unit_test', glob: '', zipFile: ZIP_FILE_NAME
						}
						
					}
				}
			}
        }
		
		stage('Summary') {
            steps {
                script {
					if(files.size() == 0){
						print("None of the three stgaes ran !")
					}else{
						print("Below Stages Ran :-")
						summary.each{ 
							print("Stage Name: "+it.name)
							print("Files copied :-")
							print(it.files)
						}
					}
				}
            }
        }
    }
	post {
	  success {
		script {
    	      try{
    	          echo "sending Email to : ${Success_Email} ...."
    	      }catch(Exception ex){
    	          echo "Couldnt send the email ...."
    	      }
	      }
	  }
	  failure {
		script {
    		try{
    	          echo "sending Email to : ${Failure_Email} ...."
    	      }catch(Exception ex){
    	          echo "Couldnt send the email ...."
    	      }
    	 }
	  }
	}
}

/*** The below function returns the list of file names read from build.json file ***/
/*** This function can kept in the shared library ***/
def build(def filePath, def zipFileName){
    def fileList = []
    def folderName = "builds"
    /*** Pipeline Utility Steps plugin is required for the below readJSON method ***/
    def items = readJSON file: filePath, text: '' // Reads the JSON file into a map.
    /*** Creates directory called "builds" if not present and create text files inside this directory ***/
    dir(folderName){
      for(file in items.files){
        if(!fileExists(file.name+".txt")){
          writeFile file: file.name+'.txt', text: file.content
        }
        fileList.add(file.name)
      }
    }
    /*** Pipeline Utility Steps plugin is required for the below zip method ***/
    zip dir: folderName, exclude: '', glob: '', zipFile: zipFileName, archive: true // creates zip file of a directory.
    return fileList
}


/*** The bleow funtion returns true if today's holiday and false if its not ***/
/*** This function can kept in the shared library ***/
def getHoliday(){
   
    def holidayToday = false
  
   /*** The hardcoded value related to the api call, api_key should be kept in jenkins credentials ***/
    def api_key = 'd20d05ccb411d9ce3b56b654971e17a29b0aa1ed'
    def country = 'IN'
  
    /*** Getting today's date in the format yyyy-mm-dd, as similar to the iso value in the json response ***/
    def now = LocalDateTime.now()
    def year = now.format(DateTimeFormatter.ofPattern("yyyy")).toString()
    def todaysDate = now.format(DateTimeFormatter.ofPattern("yyyy-MM-dd")).toString()
  
    /*** Making REST Api call using Http Request plugin in Jenkins ***/
    def response = httpRequest ignoreSslErrors: false, responseHandle: 'STRING', url: 'https://calendarific.com/api/v2/holidays?&api_key='+api_key+'&country='+country+'&year='+year, wrapAsMultipart: false
    def jsonResponse = new JsonSlurper().parseText(response.content)
    response.close()
    def holidays = jsonResponse.response.holidays
    for(holiday in holidays){
        if(holiday.date.iso == todaysDate){
            holidayToday = true
            break
        }
    }
    print("Is today Holiday : ${holidayToday.toString()}")
  
    /*** return boolean value ***/
	  return holidayToday
}
