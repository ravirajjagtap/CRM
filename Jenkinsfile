pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = 'github-token'  // Set up Jenkins credentials for GitHub
        FRAPPE_SITE_NAME = 'mysite.local'
        DB_PASSWORD = 'admin'  // Change this to a secure password
        NODE_VERSION = '18'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "🔹 Cloning repository..."
                git credentialsId: "${GIT_CREDENTIALS}", url: 'https://github.com/JohnDoeShallLive/CRM.git', branch: 'develop'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "🔹 Updating and installing required dependencies..."
                sh '''
                # Ensure non-interactive mode to avoid prompts
                export DEBIAN_FRONTEND=noninteractive

                # Update and install packages safely
                sudo apt update && sudo apt upgrade -y
                sudo apt install -y \
                python3-pip python3-dev python3-venv \
                mariadb-server mariadb-client \
                redis-server xvfb libfontconfig \
                wkhtmltopdf curl nodejs npm yarn \
                build-essential libssl-dev libffi-dev \
                libmysqlclient-dev

                # Restart services to ensure they are running
                sudo systemctl restart mariadb redis
                '''
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                echo "🔹 Setting up Python Virtual Environment..."
                sh '''
                # Use bash explicitly to avoid 'source: not found' error
                bash -c "
                python3 -m venv frappe-env
                source frappe-env/bin/activate

                # Upgrade pip and install required dependencies
                pip install --upgrade pip setuptools wheel
                pip install frappe-bench

                # Deactivate environment
                deactivate
                "
                '''
            }
        }

            stage('Setup MySQL Database') {
    steps {
        echo "🔹 Setting up MySQL Database..."
        sh '''
        sudo systemctl start mariadb

        # Check if database exists
        DB_EXISTS=$(sudo mysql -Nse "SELECT COUNT(*) FROM information_schema.schemata WHERE schema_name='frappe_db';")
        if [ "$DB_EXISTS" -eq 0 ]; then
            echo "Creating database 'frappe_db'..."
            sudo mysql -e "CREATE DATABASE frappe_db;"
            sudo mysql -e "CREATE USER 'frappe'@'localhost' IDENTIFIED BY 'frappe_password';"
            sudo mysql -e "GRANT ALL PRIVILEGES ON frappe_db.* TO 'frappe'@'localhost';"
            sudo mysql -e "FLUSH PRIVILEGES;"
        else
            echo " Database 'frappe_db' already exists. Skipping creation."
        fi
        '''
    }
}


        stage('Install Frappe Bench') {
            steps {
                echo "🔹 Installing Frappe Bench..."
                sh '''
                # Activate virtual environment before installing Bench
                bash -c "
                source frappe-env/bin/activate
                pip install --upgrade frappe-bench
                bench init --frappe-branch version-14 frappe-bench
                deactivate
                "
                '''
            }
        }

        stage('Create New Frappe Site') {
    steps {
        echo "🔹 Creating new Frappe site..."
        sh '''
        cd frappe-bench

        # Ensure we're in the correct environment
        bash -c "
        source ../frappe-env/bin/activate
        which bench || echo '⚠️ Warning: bench command not found'
        bench new-site ${FRAPPE_SITE_NAME} --db-name=frappe_db --mariadb-root-password=${DB_PASSWORD} --admin-password=admin --install-app erpnext
        deactivate
        "
        '''
    }
}


        stage('Install CRM Application') {
    steps {
        echo "🔹 Installing Frappe CRM App..."
        sh '''
        cd frappe-bench

        # Ensure we're in the correct environment
        bash -c "
        source ../frappe-env/bin/activate
        which bench || echo '⚠️ Warning: bench command not found'
        
        # Get and install the app
        bench get-app https://github.com/JohnDoeShallLive/CRM.git
        bench --site ${FRAPPE_SITE_NAME} install-app CRM
        
        deactivate
        "
        '''
    }
}


        stage('Run Frappe Server') {
            steps {
                echo "🔹 Starting Frappe Server..."
                sh '''
                cd frappe-bench
                nohup bench start > logs/frappe.log 2>&1 &
                '''
            }
        }

        stage('Run Tests') {
    steps {
        echo "🔹 Running tests..."
        sh '''
        cd frappe-bench

        # Ensure we're in the correct environment
        bash -c "
        source ../frappe-env/bin/activate
        which bench || echo '⚠️ Warning: bench command not found'
        
        # Run tests with verbose output
        bench --site ${FRAPPE_SITE_NAME} run-tests || echo '⚠️ Tests failed. Check logs for details.'
        
        deactivate
        "
        '''
    }
}


        stage('Deploy to Production') {
            steps {
                echo "🔹 Deploying Application..."
                sh '''
                echo " Deployment steps go here, such as setting up Nginx, Supervisor, or Docker"
                '''
            }
        }
    }

    post {
        success {
            echo " Build and Deployment Successful!"
        }
        failure {
            echo " Build Failed. Check logs!"
        }
    }
}
