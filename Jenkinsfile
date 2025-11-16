pipeline {
  agent any

  environment {
    DEPLOY_DIR = 'C:\\inetpub\\wwwroot\\practical1'
  }

  stages {
    stage('Checkout') {
      steps {
        echo 'Cloning project from GitHub...'
        git branch: 'master', url: 'https://github.com/Aeju011/test23.git'
      }
    }

    stage('Build') {
      steps {
        echo 'Static portfolio site – no build step needed.'
        bat 'dir'
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying portfolio files to IIS folder: ${env.DEPLOY_DIR}"
        script {
          try {
            def ws = new File(env.WORKSPACE)
            def target = new File(env.DEPLOY_DIR)

            // Create target directory if missing
            if (!target.exists()) {
              if (!target.mkdirs()) {
                error "Failed to create target folder: ${target}"
              }
              echo "Created target folder: ${target}"
            } else {
              echo "Target exists: ${target}"
            }

            // Files to copy
            def files = ['index.html','about.html','style.css']
            files.each { fname ->
              def src = new File(ws, fname)
              if (src.exists() && src.isFile()) {
                def dest = new File(target, fname)
                dest.parentFile.mkdirs()
                dest.withOutputStream { os ->
                  src.withInputStream { is ->
                    os << is
                  }
                }
                echo "Copied file: ${fname}"
              } else {
                echo "Skipping (not found): ${fname}"
              }
            }

            // Copy assets directory recursively if present
            def assetsSrc = new File(ws, 'assets')
            if (assetsSrc.exists() && assetsSrc.isDirectory()) {
              assetsSrc.eachFileRecurse { file ->
                if (file.isFile()) {
                  def rel = file.path.substring(ws.path.length() + 1)          // path relative to workspace
                  // Normalize separators to build destination path
                  rel = rel.replaceAll('\\\\', '/')
                  def dest = new File(target, rel)
                  dest.parentFile.mkdirs()
                  dest.withOutputStream { os ->
                    file.withInputStream { is ->
                      os << is
                    }
                  }
                }
              }
              echo "Copied assets folder recursively."
            } else {
              echo "No assets folder to copy."
            }

            // Print a quick listing of target folder (for debug)
            echo "Target directory listing (top-level):"
            target.listFiles()?.each { f -> echo " - ${f.name} ${f.isDirectory() ? '(dir)' : '(file)'}" }

            echo "Deployment finished."
          } catch (Exception e) {
            // print stack for debugging and fail
            echo "ERROR during deploy: ${e.toString()}"
            e.printStackTrace()
            error "Deployment failed: ${e.message}"
          }
        }
      }
    }

    stage('Run HTTP Server (IIS serves it)') {
      steps {
        echo 'IIS will serve the website. No extra server is started by Jenkins.'
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline finished successfully!'
      echo '👉 Open in browser: http://localhost/practical1/index.html'
    }
    failure {
      echo '❌ Pipeline failed! Check build logs.'
    }
  }
}
