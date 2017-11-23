// A Declarative Pipeline is defined within a 'pipeline' block.
pipeline {

  // agent defines where the pipeline will run
  // if you have a slave defined by a label you can use that - e.g. java8 or postgres or whatever
  agent {
    // This also could have been 'agent any' - that has the same meaning.
    label ""
    // Other possible built-in agent types are 'agent none', for not running the
    // top-level on any agent (which results in you needing to specify agents on
    // each stage and do explicit checkouts of scm in those stages), 'docker',
    // and 'dockerfile'.
  }
  
  // The tools directive allows you to automatically install tools configured in
  // Jenkins - note that it doesn't work inside Docker containers currently.
  //  tools {
  //    jdk "jdk8"
  //    maven "mvn3.3.8"
  //  }
  
  environment {
    // Environment variable identifiers need to be both valid bash variable
    // identifiers and valid Groovy variable identifiers. If you use an invalid
    // identifier, you'll get an error at validation time.
    // Right now, you can't do more complicated Groovy expressions or nesting of
    // other env vars in environment variable values, but that will be possible
    // when https://issues.jenkins-ci.org/browse/JENKINS-41748 is merged and
    // released.
    FOO = "BAR"
  }
  
  stages {
    // At least one stage is required.
    stage("first stage") {
      // Every stage must have a steps block containing at least one step.
      steps {
        // You can use steps that take another block of steps as an argument.
          sh "sleep 10" 
          echo "We're not doing anything particularly special here."
          echo "Hello World etc."
          echo "FOO is ${FOO}"
          sh "sleep 5" 
      }
      
      // Post can be used both on individual stages and for the entire build.
      post {
        success {
          echo "Only when we haven't failed running the first stage"
        }
        
        failure {
          echo "Only when we fail running the first stage."
        }
      }
    }
    
    stage('second stage') {
      
      steps {
        echo "This time, we could do something more interesting than a quick sleep..."
        sh "sleep 5"
        sh "printenv"
        sh "sleep 5"
      }
    }
    
    stage('third stage') {
      steps {
        // Note that parallel can only be used as the only step for a stage.
        // Also, if you want to have your parallel branches run on different
        // nodes, you'll need control that manually with "node('some-label') {"
        // blocks inside the parallel branches, and per-stage post won't be able
        // to see anything from the parallel workspaces.
        // This'll be improved by https://issues.jenkins-ci.org/browse/JENKINS-41334, 
        // which adds Declarative-specific syntax for parallel stage execution.
        parallel(one: {
                  echo "I'm on the first branch!"
                 },
                 two: {
                   echo "I'm on the second branch!"
                 },
                 three: {
                   echo "I'm on the third branch!"
                   echo "But you probably guessed that already."
                 })
      }
    }
    
    stage('testing'){
      steps{
          script {
                    def browsers = ['chrome', 'firefox', 'vivaldi', 'safari', 'tor']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                  }
      }
    }
  }
  
  post {
    // post is like the 'finally' block in Java
    // post always runs, and it runs before any of the other post conditions.
    always {
      // Let's wipe out the workspace before we finish!
      deleteDir()
    }
    
    success {
      sh "echo Yay for this, sending an email..."
      mail(from: "donaldsimpson@gmail.com", 
          to: "jpikoulas@technasthai.co.uk", 
           subject: "Build ${BUILD_NUMBER} of the ${JOB_NAME} job has passed.",
           body: "Nothing to see here, but there could be some really interesting info.")
    }

    failure {
      sh "echo Boo to that, sending an email..."
      mail(from: "donaldsimpson@gmail.com", 
           to: "jpikoulas@technasthai.co.uk", 
           subject: "Build ${BUILD_NUMBER} of the ${JOB_NAME} job has failed.",
           body: "Nothing to see here, but an error message may be useful.")
    }
  }
  
  // The options directive is for configuration that applies to the whole job.
  // More details here: https://jenkins.io/doc/book/pipeline/syntax/#options
  options {
    // For example, we'd like to make sure we only keep 10 builds at a time, so
    // we don't fill up our storage!
    buildDiscarder(logRotator(numToKeepStr:'10'))
    
    // On failure, retry the entire Pipeline the specified number of times
    retry(3)
    
    // And we'd really like to be sure that this build doesn't hang forever, so
    // let's time it out after an hour.
    timeout(time: 60, unit: 'MINUTES')
  }

}
