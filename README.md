# =========================
# ltibloodbank – Ubuntu Setup & Deployment Guide
# =========================

# 1️⃣ Setting up Ubuntu Machine
sudo apt-get update -y
sudo apt-get install apache2 -y
sudo apt-get install php libapache2-mod-php php-mysql php-curl php-gd php-json php-zip php-mbstring -y
sudo systemctl restart apache2
sudo systemctl enable apache2
sudo apt-get install mysql-server -y

# =========================
# 2️⃣ Connecting to MySQL Database
# =========================
mysql -h <DB-ENDPOINT> -u admin -p

# Create Database
CREATE DATABASE customers;
USE customers;

# Create Tables

# Donors Table
CREATE TABLE donors(
    id INT AUTO_INCREMENT PRIMARY KEY,
    fname VARCHAR(255) NOT NULL,
    lname VARCHAR(255) NOT NULL,
    mobileno BIGINT UNIQUE,
    city VARCHAR(255) NOT NULL,
    bfrom DATE,
    bto DATE,
    dob DATE,
    bloodgroup VARCHAR(255) NOT NULL
);

# Users Table
CREATE TABLE users(
    username VARCHAR(80) NOT NULL,
    name VARCHAR(80) NOT NULL,
    password VARCHAR(80) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

# Admin Table
CREATE TABLE admin(
    username VARCHAR(80) NOT NULL,
    name VARCHAR(80) NOT NULL,
    password VARCHAR(80) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

# Insert Sample Data

# Donors
INSERT INTO donors (fname, lname, mobileno, city, bfrom, bto, dob, bloodgroup) VALUES
('Srikanth', 'Koraveni', 9000736060, 'Pune', '2022-09-28', '2022-12-28', '1998-05-22', 'O_Positive'),
('Prashanth', 'Katkam', 7989919097, 'Mumbai', '2022-09-17', '2022-11-18', '1998-09-30', 'O_Positive'),
('Kranthi', 'Khaitha', 9876789871, 'Bangalore', '2022-09-16', '2022-11-08', '1996-07-02', 'B_Positive'),
('Srinivas', 'Thota', 9812789411, 'Mumbai', '2022-09-18', '2022-10-31', '1992-07-22', 'O_Positive'),
('Pandya', 'Loka', 9877787887, 'Mumbai', '2022-09-18', '2022-10-09', '1992-07-22', 'B_Positive'),
('Prajodh', 'Shreya', 9812444411, 'Mumbai', '2022-08-23', '2022-10-31', '1992-07-22', 'B_Positive'),
('Srinivas', 'Thota', 9812723411, 'Mumbai', '2022-04-19', '2022-10-07', '1992-07-22', 'B_Positive'),
('Zaheer', 'Khan', 7788678987, 'Chennai', '2022-09-11', '2022-12-19', '1998-11-11', 'A_Positive');

# Users
INSERT INTO users (username, name, password) VALUES
('yssyogesh', 'Yogesh Singh', '12345'),
('bsonarika', 'Sonarika Bhadoria', '12345'),
('vishal', 'Vishal Sahu', '12345'),
('prashanth', 'Prashanth Katkam', '12345'),
('vijay', 'Vijay mourya', '12345');

# Admin
INSERT INTO admin (username, name, password) VALUES
('admin', 'admin', '12345');

# Grant Permissions
GRANT ALL PRIVILEGES ON customers.* TO 'root'@'%' IDENTIFIED BY 'admin123';
FLUSH PRIVILEGES;

# =========================
# 3️⃣ Important Notes
# =========================
# - If EC2 cannot connect to DB, add inbound rule to DB security group (SG) and attach EC2 SG.
# - Add DB endpoint in PHP files:
#   config.php, donate-blood.php, find-donor.php, search.php, signup.php, deletedata.php
# - Ensure table names match in PHP files:
#   donors -> index.php
#   admin -> indexadmin.php

# =========================
# 4️⃣ Git Commands
# =========================
git clone <repo-URL>
git clone --branch <branch-name> <repo-URL>

git init
git remote add origin <repo-URL>
git add .
git commit -m "Initial commit"
git push origin master

# ⚠ Do not commit secrets (like GitHub PATs) to repo.

# =========================
# 5️⃣ Install Jenkins
# =========================
sudo apt-get update
sudo apt-get install openjdk-8-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
sudo apt install git

# =========================
# 6️⃣ Push Apache Logs to CloudWatch
# =========================
# 1. Create EC2 and attach role with CloudWatchAgentServerPolicy
# 2. Update instance
sudo apt-get update
# 3. Install Apache
sudo apt-get install apache2
# 4. Download and install CloudWatch Agent
sudo wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb

# 5. Configure CloudWatch
vi /opt/aws/amazon-cloudwatch-agent/bin/config.json
# Example config.json:
# {
#   "agent": {"run_as_user": "root"},
#   "logs": {
#     "logs_collected": {
#       "files": {
#         "collect_list": [
#           {
#             "file_path": "/var/log/apache2/access.log",
#             "log_group_name": "myapache-error-log",
#             "log_stream_name": "{instance_id}"
#           }
#         ]
#       }
#     }
#   }
# }

# 6. Start CloudWatch Agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

# =========================
# 7️⃣ Useful SQL Commands
# =========================
DELETE FROM customers;
SHOW COLUMNS FROM donors;

# =========================
# 8️⃣ Sample HTML Page
# =========================
cat <<EOL > index.html
<html>
  <body>
    Hi, this is the webpage after making changes in GitHub repo and CI/CD is visible here.
  </body>
</html>
EOL
