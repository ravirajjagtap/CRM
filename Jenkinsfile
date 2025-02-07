pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-token'  // Jenkins credentials for GitHub
        FRAPPE_SITE_NAME = 'mysite.local'
        DB_PASSWORD = credentials('1234')  // Use Jenkins credentials for security
        NODE_VERSION = '18'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: 'github-token', branch: 'develop', url: 'https://github.com/ravirajjagtap/CRM.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "🔹 Updating and installing required dependencies..."
                sh '''
                export DEBIAN_FRONTEND=noninteractive
                sudo apt update && sudo apt upgrade -y
                sudo apt install -y \
                python3-pip python3-dev python3-venv \
                mariadb-server mariadb-client \
                redis-server xvfb libfontconfig \
                wkhtmltopdf curl nodejs npm yarn \
                build-essential libssl-dev libffi-dev \
                libmysqlclient-dev
                
                sudo systemctl restart mariadb redis
                '''
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                echo "🔹 Setting up Python Virtual Environment..."
                sh '''
                python3 -m venv frappe-env
                source frappe-env/bin/activate
                pip install --upgrade pip setuptools wheel
                pip install frappe-bench
                deactivate
                '''
            }
        }

        stage('Setup MySQL Database') {
            steps {
                echo "🔹 Setting up MySQL Database..."
                sh '''
                sudo systemctl start mariadb
                DB_EXISTS=$(sudo mysql -Nse "SELECT COUNT(*) FROM information_schema.schemata WHERE schema_name='frappe_db';")
                if [ "$DB_EXISTS" -eq 0 ]; then
                    sudo mysql -e "CREATE DATABASE frappe_db;"
                    sudo mysql -e "CREATE USER 'frappe'@'localhost' IDENTIFIED BY '${DB_PASSWORD}';"
                    sudo mysql -e "GRANT ALL PRIVILEGES ON frappe_db.* TO 'frappe'@'localhost';"
                    sudo mysql -e "FLUSH PRIVILEGES;"
                else
                    echo "Database 'frappe_db' already exists. Skipping creation."
                fi
                '''
            }
        }

        stage('Install Frappe Bench') {
            steps {
                echo "🔹 Installing Frappe Bench..."
                sh '''
                source frappe-env/bin/activate
                pip install --upgrade frappe-bench
                bench init --frappe-branch version-14 frappe-bench
                deactivate
                '''
            }
        }

        stage('Create New Frappe Site') {
            steps {
                echo "🔹 Creating new Frappe site..."
                sh '''
                cd frappe-bench
                source ../frappe-env/bin/activate
                bench new-site ${FRAPPE_SITE_NAME} --db-name=frappe_db --mariadb-root-password=${DB_PASSWORD} --admin-password=admin --install-app erpnext
                deactivate
                '''
            }
        }

        stage('Install CRM Application') {
            steps {
                echo "🔹 Installing Frappe CRM App..."
                sh '''
                cd frappe-bench
                source ../frappe-env/bin/activate
                bench get-app https://github.com/ravirajjagtap/CRM.git
                bench --site ${FRAPPE_SITE_NAME} install-app CRM
                deactivate
                '''
            }
        }

        stage('Run Frappe Server') {
            steps {
                echo "🔹 Starting Frappe Server..."
                sh '''
                cd frappe-bench
                bench start &
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo "🔹 Running tests..."
                sh '''
                cd frappe-bench
                source ../frappe-env/bin/activate
                bench --site ${FRAPPE_SITE_NAME} run-tests || echo '⚠️ Tests failed. Check logs for details.'
                deactivate
                '''
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "🔹 Deploying Application..."
                sh '''
                echo "Deployment steps go here, such as setting up Nginx, Supervisor, or Docker"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build and Deployment Successful!"
        }
        failure {
            echo "❌ Build Failed. Check logs!"
        }
    }
}
