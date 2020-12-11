node {
	checkout scm

	def jobs = [:]

	toxenv = sh(script: 'tox --listenvs', returnStdout: true).trim().split('\n')
	toxenv.each {
		item ->
			jobs[item] = {
				stage('Test') {
					sh(script: 'export TMPDIR=/var/tmp; tox -r -e ' + item + ' test')
				}
			}
	}

	parallel(jobs)
}
