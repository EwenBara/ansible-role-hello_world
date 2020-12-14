node {
	checkout scm

	def jobs = [:]

	toxenv = sh(script: 'tox --listenvs', returnStdout: true).trim().split('\n')
	toxenv.each {
		item ->
			jobs[item] = {
				try {
					stage('Preprare venv') {
							sh(script: 'export TMPDIR=/var/tmp; tox -r --notest -e ' + item)
					}
				}
				catch(e) {
					currentStage.result = 'NOT_BUILT'
					currentBuild.result = 'FAILURE'
					throw e
				}
				try {
					try {
						stage('Lint') {
							sh(script: 'tox -e ' + item + ' -- lint')
						}
						stage('Syntax') {
							sh(script: 'tox -e ' + item + ' -- syntax')
						}
					}
					catch(e) {
						currentStage.result = 'FAILURE'
						currentBuild.result = 'FAILURE'
						throw e
					}
					try {
						stage('Create') {
							sh(script: 'tox -e ' + item + ' -- syntax')
						}
					}
					catch(e) {
						currentStage.result = 'FAILURE'
						currentBuild.result = 'NOT_BUILT'
						throw e
					}
					try {
						stage('Converge') {
							sh(script: 'tox -e ' + item + ' -- converge')
						}
					}
					catch(e) {
						currentStage.result = 'FAILURE'
						currentBuild.result = 'FAILURE'
						throw e
					}
					try {
						stage('Idempotence') {
							sh(script: 'tox -e ' + item + ' -- idempotence')
						}
					}
					catch(e) {
						currentStage.result = 'FAILURE'
						currentBuild.result = 'UNSTABLE'
						throw e
					}
					try {
						stage('Verify') {
							sh(script: 'tox -e ' + item + ' -- idempotence')
						}
					}
					catch(e) {
						currentStage.result = 'FAILURE'
						currentBuild.result = 'FAILURE'
						throw e
					}
				}
				finally {
					stage('Clean') {
						sh(script: 'tox -e ' + item + ' -- destroy')
					}
				}
			}
	}

	parallel(jobs)
}
