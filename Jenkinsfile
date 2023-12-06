pipeline {
    agent any

    parameters {
        choice(
            choices: ['dev', 'staging', 'prod'],
            description: 'Select the target server.',
            name: 'TARGET_SERVER'
        )

        string(
            defaultValue: '',
            description: 'Database name to dump.',
            name: 'DB_NAME'
        )
    }

    stages {
        stage('Connect to Server') {
            steps {
                script {
                    def targetEnv = params.TARGET_SERVER
                    def targetServer
                    def dbName = params.DB_NAME
                    def remoteCommand
                    def commandArguments = "--skip_add_locks --skip-lock-tables --single-transaction --lock-for-backup --triggers --events --routines ${dbName} | gzip -9 > ${dbName}.sql.gz"

                    if (targetEnv == 'dev') {
                        targetServer = '168.119.154.205'

                    } else if (targetEnv == 'staging') {
                        targetServer = '188.34.161.26'

                    } else if (targetEnv == 'prod') {
                        targetServer = '159.65.107.206'

                    } else {
                        error "Invalid environment specified"
                    }

                    if (targetEnv == 'dev') {
                        def commandExitCode = sh(script: "mysql --defaults-group-suffix=5 -e \"show databases\" | grep ${dbName}", returnStatus: true)

                        remoteCommand = (commandExitCode == 0) ? "mysqldump --defaults-group-suffix=5 ${commandArguments}" : "mysqldump --defaults-group-suffix=8 ${commandArguments}"

                    } else if (targetEnv == 'staging' || targetEnv == 'prod') {
                        remoteCommand = "mysqldump ${commandArguments}"

                    } else {
                        error "Invalid server specified"
                    }

                    sshagent(['de3b4309-6c71-4178-98ac-d66e98a1964d	']) {
                        sh "ssh api@${targetServer} '${remoteCommand}'"
                    }
                }
            }
        }
    }
}