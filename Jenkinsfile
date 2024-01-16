pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        script {
          // Test Case 1: Automated Testing in CI
          sh 'docker build -t my-flask .'
          sh 'docker tag my-flask $DOCKER_BFLASK_IMAGE'
      
          // Run tests and fail the pipeline if any tests fail
          sh 'docker run my-flask python -m pytest app/tests/' || error("Unit tests failed.")
        }
      }
    }

    
    stage('Nightly Build') {
      // Test Case 2: Scheduled Builds
      triggers {
        cron('H 0 * * *') // Schedule nightly build at midnight
      }
      steps {
        script {
          sh 'docker build -t my-flask .'
          sh 'docker tag my-flask $DOCKER_BFLASK_IMAGE'
        }
      }
    }
    
    stage('Deploy') {
      steps {
        script {
          // Test Case 3: Rollback Mechanism in CD
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            // Deploy faulty version intentionally
            sh 'docker push $FAULTY_DOCKER_BFLASK_IMAGE'
          }
          // Trigger rollback if deployment fails
          catchError(buildResult: 'FAILURE') {
            input 'Rollback to the previous version?'
            sh 'docker pull $DOCKER_BFLASK_IMAGE'
            sh 'docker tag $DOCKER_BFLASK_IMAGE $FAULTY_DOCKER_BFLASK_IMAGE'
            sh 'docker push $FAULTY_DOCKER_BFLASK_IMAGE'
          }
        }
      }
    }
  }
}
