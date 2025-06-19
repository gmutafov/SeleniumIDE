pipeline {
    agent any

    environment {
        CHROME_VERSION = '137.0.7151.7000'
        CHROMEDRIVER_VERSION = '137.0.7151.7000'
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROME_DRIVER_PATH = 'C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Set up .NET Core') {
            steps {
                bat '''
                    echo Installing .NET SDK 6.0
                    choco install dotnet-sdk -y --version=6.0.100
                '''
            }
        }

        stage('Uninstall Current Chrome') {
            steps {
                bat '''
                    echo Attempting to uninstall Google Chrome (if installed)
                    choco uninstall googlechrome -y || echo "Chrome not installed, skipping uninstall"
                '''
            }
        }


        stage('Install Specific Chrome Version') {
            steps {
                bat '''
                    echo Installing Google Chrome version %CHROME_VERSION%
                    choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums
                '''
            }
        }



        stage('Restore Dependencies') {
            steps {
                bat 'dotnet restore SeleniumIde.sln'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build SeleniumIde.sln --configuration Release'
            }
        }

        stage('Run Tests') {
            steps {
                bat '''
                    mkdir TestResults
                    dotnet test SeleniumIde.sln --logger "trx;LogFileName=TestResults\\TestResults.trx"
                '''
            }
        }

        stage('Convert TRX to JUnit XML') {
            steps {
                bat '''
                    echo Installing trx2junit globally
                    dotnet tool install --global trx2junit || echo "Already installed"

                    echo Converting TRX to JUnit XML
                    trx2junit TestResults\\TestResults.trx

                    dir TestResults
                '''
            }
        }


    }


    post {
        always {
            junit 'TestResults/*.xml'
        }
    }

}