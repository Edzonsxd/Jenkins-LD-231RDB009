// --- FUNKCIJAS ---
def installDependencies() {
    echo "Installing all required dependencies for Python virtual environment..."
    dir('python-greetings') {
        git url: 'https://github.com/mtararujs/python-greetings.git', branch: 'main'

        bat '''
            python -m venv venv
            venv\\Scripts\\python.exe -m pip install -r requirements.txt
        '''
    }
}

def deployApp(envName, port) {
    echo "Deploying application to the ${envName.toUpperCase()} environment on port ${port}..."
    dir('python-greetings') {
        git url: 'https://github.com/mtararujs/python-greetings.git', branch: 'main'

        bat """
            call pm2 delete greetings-app-${envName} >NUL 2>&1
            exit /B 0
        """

        bat """
            set JENKINS_NODE_COOKIE=dontKillMe
            call pm2 start app.py --name greetings-app-${envName} --interpreter "%WORKSPACE%\\python-greetings\\venv\\Scripts\\python.exe" -- --port ${port}
        """

        // Servisa pieejamības pārbaude
        bat """
            setlocal EnableDelayedExpansion
            set "READY=0"
            for /L %%i in (1,1,30) do (
                curl -fsS http://127.0.0.1:${port}/greetings | findstr /C:"Hello from Python App!" >NUL 2>&1 && (
                    set "READY=1"
                    goto :ready
                )
                ping 127.0.0.1 -n 2 >NUL 2>&1
            )
            :ready
            if "!READY!"=="1" (
                echo Service is ready on port ${port}
                exit /b 0
            )
            echo Service did not become ready on port ${port}
            call pm2 logs greetings-app-${envName} --lines 50 --nostream
            exit /b 1
        """
    }
}

def runTests(envName) {
    echo "Running API tests on the ${envName.toUpperCase()} environment..."
    dir('course-js-api-framework') {
        git url: 'https://github.com/mtararujs/course-js-api-framework.git', branch: 'main'

        bat 'call npm install'

        bat "call npm run greetings -- greetings_${envName}"
    }
    
    bat """
        call pm2 delete greetings-app-${envName} >NUL 2>&1
        exit /B 0
    """
}

// --- GALVENAIS PIEGĀDES KONVEIJERS ---
pipeline {
    agent any
    stages {
        stage('install-pip-deps') {
            steps {
                script { installDependencies() } 
            }
        }
        
        stage('deploy-to-dev') {
            steps {
                script { deployApp('dev', '7001') }
            }
        }
        stage('tests-on-dev') {
            steps {
                script { runTests('dev') }
            }
        }
        
        stage('deploy-to-stg') {
            steps {
                script { deployApp('stg', '7002') }
            }
        }
        stage('tests-on-stg') {
            steps {
                script { runTests('stg') }
            }
        }
        
        stage('deploy-to-preprod') {
            steps {
                script { deployApp('preprod', '7003') }
            }
        }
        stage('tests-on-preprod') {
            steps {
                script { runTests('preprod') }
            }
        }
        
        stage('deploy-to-prod') {
            steps {
                script { deployApp('prod', '7004') }
            }
        }
        stage('tests-on-prod') {
            steps {
                script { runTests('prod') }
            }
        }
    }
}