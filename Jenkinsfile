stage('CI') {
    node('master') {
        try {
            checkout scm
            // git repo and branch defined inside Jenkins job
            
            sh label: '',
                script: 'npm install'
                
            stash name: 'everything',
                    excludes: 'test-results/**',
                    includes: '**'
            
            sh 'npm run test-single-run -- --browsers PhantomJS'
            
            step([$class: 'JUnitResultArchiver',
                    testResults: 'test-results/**/test-results.xml'])
            
            notify('Success')
        } catch(err) {
            notify("Build failed: ${err}")
            currentBuild.result = 'FAILURE'
        }
    }
}

// parallel integration testing
stage('Browser-Testing') {
    parallel chrome: {
        runTests("Chrome")
    },
    firefox: {
        runTests("Firefox")
    }
}

node {
    notify("Deploy to staging?")
}

input message: 'Deploy to staging?'

stage('Deploy to staging') {
    node {
        // write build number to index page so we can see this update
        // on windows use: bat "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
        sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
        
        // deploy to a docker container mapped to port 3000
        // on windows use: bat 'docker-compose up -d --build'
        sh 'docker-compose up -d --build'
        
        notify 'Solitaire Deployed!'
    }
}

def runTests(browser) {
    node {
        sh 'rm -rf *'
        unstash name: 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver',
                    testResults: 'test-results/**/test-results.xml'])
    }
}

def notify(status){
    emailext (
        from: "jenkins-server@secret.garden",
        to: "adhishm@gmail.com",
        subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
        body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}

