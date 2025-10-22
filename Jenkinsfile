pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '20'))
    disableConcurrentBuilds()
  }

  /* ====== PARAMETER YANG BISA DIUBAH SAAT RUN ====== */
  parameters {
    string(name: 'TARGET_BASE_URL', defaultValue: 'https://tst.yokke.co.id:3524', description: 'Base URL target test')
    string(name: 'THREADS',         defaultValue: '100', description: 'Jumlah virtual users')
    string(name: 'RAMP_UP',         defaultValue: '120', description: 'Detik ramp-up')
    string(name: 'DURATION',        defaultValue: '300', description: 'Detik durasi (Scheduler)')
    booleanParam(name: 'SCHEDULED', defaultValue: true, description: 'Aktifkan scheduler di JMX?')
  }

  /* ====== SESUAIKAN PATH DI BAWAH INI DENGAN AGENT WINDOWS KAMU ====== */
  environment {
    JMETER_HOME = 'D:\\TAP\\jmeter'                            // harus berisi \\bin\\jmeter.bat
    TEST_PLAN   = 'jmeter\\testplans\\IMS-Project.jmx'         // path relatif di repo
    OUT_DIR     = "reports\\${env.BUILD_NUMBER}"               // output per-build
    JTL_FILE    = "${env.OUT_DIR}\\results.jtl"
    HTML_DIR    = "${env.OUT_DIR}\\html"
  }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Prepare') {
      steps {
        bat """
        if not exist "${OUT_DIR}" mkdir "${OUT_DIR}"
        if not exist "${HTML_DIR}" mkdir "${HTML_DIR}"
        """
      }
    }

    stage('Run JMeter (non-GUI)') {
      steps {
        // -J property akan dibaca di JMX via ${__P(name,default)}
        bat """
        call "${JMETER_HOME}\\bin\\jmeter.bat" -n ^
          -t "${TEST_PLAN}" ^
          -l "${JTL_FILE}" ^
          -e -o "${HTML_DIR}" ^
          -JbaseUrl=${params.TARGET_BASE_URL} ^
          -Jthreads=${params.THREADS} ^
          -Jrampup=${params.RAMP_UP} ^
          -Jduration=${params.DURATION} ^
          -Jschedule=${params.SCHEDULED}
        """
      }
    }

    stage('Archive & Publish Reports') {
      steps {
        // Arsipkan semua artefak report
        archiveArtifacts artifacts: "${OUT_DIR}/**", fingerprint: true

        // Tampilkan HTML Dashboard di Jenkins
        publishHTML(target: [
          allowMissing: false,
          alwaysLinkToLastBuild: false,
          keepAll: true,
          reportDir: "${HTML_DIR}",
          reportFiles: "index.html",
          reportName: "JMeter Dashboard (Build ${env.BUILD_NUMBER})"
        ])

        // Kirim .jtl ke Performance Plugin untuk grafik tren
        perfReport errorFailedThreshold: 0,
                   errorUnstableThreshold: 0,
                   sourceDataFiles: "${JTL_FILE}"
      }
    }
  }

  post {
    always {
      echo "Artifacts: ${env.BUILD_URL}artifact/${OUT_DIR}/"
    }
    // Aktifkan kalau SMTP sudah diset
    /*
    success {
      emailext subject: "OK: JMeter build #${env.BUILD_NUMBER}",
               to: 'team-qa@example.com',
               body: "Build sukses. Report HTML: ${env.BUILD_URL}JMeter_20Dashboard_20(Build_20${env.BUILD_NUMBER})/"
    }
    failure {
      emailext subject: "FAILED: JMeter build #${env.BUILD_NUMBER}",
               to: 'team-qa@example.com',
               body: "Build gagal. Console log: ${env.BUILD_URL}console"
    }
    */
  }
}