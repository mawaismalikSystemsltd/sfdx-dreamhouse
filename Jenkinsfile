#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME
    def PACKAGE_NAME = '00D5j000007dYrCEAU'
    def TEST_LEVEL='RunLocalTests'

    def HUB_ORG = "awaisafiniti@empathetic-wolf-tel0yq.com" //"mawais.malik@empathetic-unicorn-9xl702.com" //env.HUB_ORG_DH
    def SFDC_HOST = "https://login.salesforce.com" //env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = "660eb69d-a259-4ba6-a533-cfc0e75c24e3" //"JWT_KEY_FILE" //env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = "3MVG9pRzvMkjMb6lstWCxYFG6iUPAHDa8R232L0c3UuOdo_qfCStwJRqFprFf03NdcRyvEpX8oj559R8Z_jCf" //"3MVG9pRzvMkjMb6ldL9vVDPBGsdJIUFY.DLZsh8lJcWBI_V2L5O6HI.tjFmCt2zOWwKVXcZTlWPsKarOJmKto" //env.CONNECTED_APP_CONSUMER_KEY_DH

    
    println 'KEY IS'
    println JWT_KEY_CRED_ID
    println 'ORG IS'
    println HUB_ORG
    println 'HOST IS'
    println SFDC_HOST
    println 'APP_KEY IS'
    println CONNECTED_APP_CONSUMER_KEY
    
    
    
    //def sfdxTool
    def toolbelt = tool 'sfdx'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    
     withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize DevHub') {
                rc = command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias HubOrg"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Create new scratch org to your code.
            // -------------------------------------------------------------------------

            stage('Create Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce test scratch org creation failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Display scratch org info.
            // -------------------------------------------------------------------------

            stage('Display Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:display --targetusername ciorg"
                if (rc != 0) {
                    error 'Salesforce test scratch org display failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Push source to scratch org.
            // -------------------------------------------------------------------------

            stage('Push To Scratch Org') {
                rc = command "${toolbelt}/sfdx force:source:push --targetusername ciorg  " //--all --targetusername ciorg -y debug
                if (rc != 0) {
                    error 'Salesforce push to test scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Run unit tests in scratch org.
            // -------------------------------------------------------------------------

            stage('Run Tests In Scratch Org') {
                rc = command "${toolbelt}/sfdx force:apex:test:run --targetusername ciorg --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
                if (rc != 0) {
                    error 'Salesforce unit test run in test scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Delete scratch org.
            // -------------------------------------------------------------------------

            stage('Delete Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:delete --targetusername ciorg --noprompt"
                if (rc != 0) {
                    error 'Salesforce test scratch org deletion failed.'
                }
            }

            // -------------------------------------------------------------------------
            // collect results.
            // -------------------------------------------------------------------------
	
			stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
			}
            
        }
    }

}
def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }
}
    
