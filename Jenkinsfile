node {
	checkout scm

	def jobs = [:]
	def jobsResult = [:]

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
					jobsResult[item] = 'NOT_BUILT'
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
						jobsResult[item] = 'FAILURE'
						throw e
					}
					try {
						stage('Create') {
							sh(script: 'tox -e ' + item + ' -- syntax')
						}
					}
					catch(e) {
						jobsResult[item] = 'NOT_BUILT'
						throw e
					}
					try {
						stage('Converge') {
							sh(script: 'tox -e ' + item + ' -- converge')
						}
					}
					catch(e) {
						jobsResult[item] = 'FAILURE'
						throw e
					}
					try {
						stage('Idempotence') {
							sh(script: 'tox -e ' + item + ' -- idempotence')
						}
					}
					catch(e) {
						jobsResult[item] = 'UNSTABLE'
						throw e
					}
					try {
						stage('Verify') {
							sh(script: 'tox -e ' + item + ' -- verify')
							jobsResult[item] = 'SUCCESS'
						}
					}
					catch(e) {
						jobsResult[item] = 'FAILURE'
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

	if(jobsResult.count('SUCCESS') == 0) {
		currentBuild.result == 'FAILURE'
	} else if(jobsResult.count('SUCCESS') != jobsResult.size()) {
		currentBuild.result == 'UNSTABLE'
	}
}
