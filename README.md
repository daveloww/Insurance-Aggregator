# Insurance Aggregator

## Table of contents
1. [Introduction](#Introduction)
2. [Technical Overview](#Technical-Overview)
    1. [Interface Layer](#Interface-Layer)
    2. [Composite Service Layer](#Composite-Service-Layer)
    3. [Atomic Service Layer](#Atomic-Service-Layer)
    4. [Database Layer](#Database-Layer)
3. [Prerequisites](#Prerequisites)
4. [Getting Started](#Getting-Started)
    1. [Clone Repository](#Clone-Repository)
    2. [Database Setup](#Database-Setup)
    3. [SendGrid Setup](#SendGrid-Setup)
    4. [Paypal Setup](#Paypal-Setup)
    5. [Environment Check](#Environment-Check)
5. [Running the Application](#Running-the-Application)
    1. [Start Docker Containers](#Start-Docker-Containers)
    2. [Configure Kong API Gateway](#Configure-Kong-API-Gateway)
6. [Navigating the Application](#Navigating-the-Application)
    1. [Customer](#Customer)
    2. [Agent](#Agent)
7. [Future Work](#Future-Work)

<br>

## Introduction
With many insurance companies offering a multitude of policies, customers may find it difficult to find a policy that best suits their needs. But fret not, this Insurance Aggregator comes to the rescue! It aggregate policies from various insurance companies and through a simple scoring algorithm, it recommends the top 5 policies that are best suited for the customer based on his/her preferences and personal information.

This repository contains instructions of setting up and running the application on localhost, meant for **Windows** or **Mac** OS. Ensure that you also have all the [prerequisites](#Prerequisites) covered before you [get started](#Getting-Started).

[Back To The Top](#Insurance-Aggregator)

<br>

## Technical Overview
The application follows a [microservices architecture](https://en.wikipedia.org/wiki/Microservices) where microservices communicate through [HTTP requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) (synchronous) and [RabbitMQ message queues](https://www.rabbitmq.com/) (asynchronous). **HTTP communication technology** is mostly used as most processes require immediate or simply a response in order to continue. In contrast, for processes such as triggering an email to the customer or to log user activities, an immediate response is not required and therefore **messaged-based asynchronous communication** is used. 

Each microservice is containerised using [Docker](https://www.docker.com/), and [Kong API Gateway](https://konghq.com/) is implemented for routing purposes to avoid exposing internal endpoints. The service-oriented architecture comprises of multiple layers - [Interface](#Interface-Layer), [Composite Service](#Composite-Service-Layer), [Atomic Service](#Atomic-Service-Layer) and [Database](#Database-Layer).

### Interface Layer
- **Customer UI**
  - index.html
  - login.html
  - registration.html
  - categories.html
  - form.html
  - policies.html
  - payment.html
  - receipt.html

- **Agent UI**
  - index.html
  - login.html
  - agent.html
  - view-customer.html

**Technologies**: HTML, CSS, JavaScript

<br>

## Composite Service Layer
- **Account** (account.py)
- **Checkout** (checkout.py)
- **Payment** (payment.py)
- **Policy** (policy.py)
- **Recommendation** (recommendation.py)
- **View Agent Customers** (view_agent_custs.py)

**Technologies**: [Python Flask](https://flask.palletsprojects.com/en/2.2.x/), [pika](https://pika.readthedocs.io/en/stable/), [RabbitMQ](https://www.rabbitmq.com/), [paypalrestsdk](https://developer.paypal.com/home)

<br>

## Atomic Service Layer
- **Agent** (agent.py)
- **Customer** (customer.py)
- **Transaction** (transaction.py)
- **Notification** (notification.py)
- **AIA** - pseudo external API (aia.py)
- **Aviva** - pseudo external API (aviva.py)
- **Great Eastern** - pseudo external API (great_eastern.py)

**Technologies**: [Python Flask](https://flask.palletsprojects.com/en/2.2.x/), [Flask SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/), [sendgrid API](https://docs.sendgrid.com/for-developers/sending-email/v3-python-code-example)

<br>

## Database Layer
- **Agent** (agent.sql)
- **Customer** (customer.sql)
- **Transaction** (transaction.sql)

**Technologies**: [MySQL](https://www.mysql.com/)

[Back To The Top](#Insurance-Aggregator)

<br>

## Prerequisites
- [Python](https://www.python.org/downloads/) >= 3.8
- [Git](https://git-scm.com/downloads) >= 2.21
- [Docker Desktop](https://docs.docker.com/get-docker/)
- [WAMP](https://www.wampserver.com/en/download-wampserver-64bits/) (Windows) / [MAMP](https://www.mamp.info/en/downloads/) (Mac)
- [Paypal](https://developer.paypal.com/tools/sandbox/accounts/#create-a-personal-sandbox-account) Account Credentials and Keys
- [SendGrid](https://signup.sendgrid.com/) email configuration and API Key

[Back To The Top](#Insurance-Aggregator)

<br>

## Getting Started
To get things started, you need to:
1. [Clone this repository](#Clone-Repository) into your local machine
2. [Set up database](#Database-Setup)
3. [Set up SendGrid](#SendGrid-Setup)
4. [Set up Paypal](#Paypal-Setup)
5. [Check environment](#Environment-Check)

<br>

### Clone Repository
Since the UI will be hosted on WAMP/MAMP, please clone the repository into the WAMP/MAMP root directory.
- Windows: **www** directory
- Mac: **htdocs** directory

<br>

Using your command-line interface/terminal window, navigate to the WAMP/MAMP root directory and run the following command:
```
git clone https://github.com/daveloww/Insurance-Aggregator.git
```
You may be prompted to log in to a Github account. Please log in to your Github account and you will be able to access this public repository.

<br>

### Database Setup
1. Start WAMP/MAMP
2. Launch your web browser and log in to [phpMyAdmin](http://localhost/phpmyadmin/)
3. Check the port number that phpMyAdmin is running on. You can check the port number via the homepage after you've logged in. The port number is shown at the top level corner, beside the logo, in the form of "**Server: MySQL: *XXXX***". ***XXXX*** is the port number
4. If the port number is 3306, please proceed to the next step. Otherwise:
   - Launch an IDE of your choice
   - Navigate to the "Insurance-Aggregator" directory ([previously cloned](#Clone-Repository)) and open the "docker-compose.yml" file
   - Press "Ctrl + F" on your keyboard and search for "3306"
   - There should be 3 search results. Replace all "3306" with your port number
5. At the homepage:
   - Click the "Import" tab
   - Click the "Choose File" button
   - Navigate to WAMP/MAMP root directory > "Insurance-Aggregator" directory > "agent.sql" file
   - Click the "Go" button at the bottom
   - Upon completion, you should see a green prompt stating that "*Import has been successfully finished..*" and the newly created database schema "agent" should appear at the left
   - Go back to the homepage
   - Repeat the step 5 two times to import "customer.sql" and "transaction.sql"
6. As MySQL restricts 'root' access of database from a remote machine (container), you are required to create a new account to grant remote access permission. Steps:
   - Log in and navigate to the homepage of phpMyAdmin
   - At the top, click on the "User accounts" tab
   - Click "Add user account" at the bottom and specify the following:
     - User name: (Use text field) **is213**
     - Host name: (Any host) **%**
     - Password: (No Password)
     - Select the "**Data**" checkbox
     - Click the "Go" button 

<br>

### SendGrid Setup
1. Log in to your account at SendGrid and copy your API key
2. Launch an IDE of your choice
3. Navigate to the "Insurance-Aggregator" directory and open the "docker-compose.yml" file
4. In line 266, paste the API key
5. Navigate to the "Insurance-Aggregator" directory and open the "notification.py" file
6. In line 75, enter the email address that is linked with the API key

<br>

### Paypal Setup
1. Log in to your account at Paypal and copy the sandbox client_id and client_secret credentials
2. Launch an IDE of your choice
3. Navigate to the "Insurance-Aggregator" directory and open the "payment.py" file
4. In line 20 and 21, paste the credentials accordingly

<br>

### Environment Check
Launch Docker Desktop and ensure that none of your existing containers are using the following ports:
- 1337
- 5432
- 5672
- 8000
- 15672

<br>

If you do, please shut down the containers and you're now ready to [run the application](#Running-The-Application)!

[Back To The Top](#Insurance-Aggregator)

<br>

## Running the Application
Please ensure that you have fulfilled the [prerequisites](#Prerequisites) and accomplished the steps in [getting started](#Getting-Started) above before you proceed.

<br>

To get a functioning application, you need to:
1. [Start Docker Containers](#Start-Docker-Containers)
2. [Configure Kong API Gateway](#Configure-Kong-API-Gateway)

<br>

### Start Docker Containers 
Using your command-line interface/terminal window, navigate to the "Insurance-Aggregator" directory ([previously cloned](#Clone-Repository)). After which, run the following command to build the Docker containers:
```
docker-compose build
```

<br>

Run the following command to start the Docker containers and run them in the background:
```
docker-compose up -d
```

<br>

#### Additional Information

To **stop** the Docker containers, press ```Ctrl``` or ```Cmd``` + ```C```. Alternatively, run the following command:
```
docker-compose down
```
<br>

To stop and remove containers, networks, images, and volumes - run the following command:
```
docker-compose down -v
```

<br>

### Configure Kong API Gateway
1. Launch your web browser and type "[localhost:1337](http://localhost:1337)"
2. Create the administrator account with the following details:
    - Username: admin
    - Email: *enter your email address*
    - Password: adminadmin
    - Confirm password: adminadmin
3. Login using the username and password above
4. Set up the Connection to Kong Admin using the following details:
    - Name: default
    - Kong Admin URL: http://kong:8001
5. Click on "SNAPSHOTS" at the left menu bar
6. Click "IMPORT FROM FILE"
7. Navigate to the "Insurance-Aggregator" directory ([previously cloned](#Clone-Repository))
8. Select and import the "konga_snapshot.json" file
9. Click on "DETAILS"
10. Click on the "RESTORE" green button
11. Click on the checkbox for "services" and leave the rest unchecked
12. Click the "IMPORT OBJECTS" button
    - You should see that 9 services are imported and 0 has failed
13. Close the modal
14. Repeat step 10
15. Click on the checkbox for "routes" and leave the rest unchecked
16. Repeat step 12 & 13

<br>

Now, start WAMP/MAMP, launch your web browser, and type in [http://localhost/Insurance-Aggregator/templates](http://localhost/Insurance-Aggregator/templates) and you may start [navigating the application](#Navigating-The-Application)!

[Back To The Top](#Insurance-Aggregator)

<br>

## Navigating the Application

There are 2 different user types - [Customer](#customer) and [Agent](#agent). 

<br>

### Customer
Customers will need to first register an account, then the application would retrieve the customer's details from SingPass Sandbox API and store it in the customer's database. 

#### Registration
- NRIC - choose from one of the dummy NRICs stated below:
  - S9812381D
  - S9812382B
  - S9812385G
  - S9812387C
  - S9912363Z
  - S9912366D
  - S9912369I
  - S9912370B
  - S9912372I
  - S9912374E
  - S6005053H
  - S6005055D

- Password: No requirements

- Email address: Your email address (you need to use a real email address to receive an email after a purchase)

<br>

Upon successful registration, you will be redirected to the login page. You may then log in with the new customer account and explore the features for customers!

<br>

Features include:
- Stating preferences (e.g. insurance category and coverage)
- Being recommended suitable policies based on stated preferences
- Using paypal to purchase a recommended policy
- Receiving an email after successful pruchase

<br>

### Agent
Agents do not need to register for an account, as dummy agent accounts already exist in the database.

*Dummy Agent Account* <br>
*NRIC: S1234567C* <br>
*Password: dave123* <br>

Use the above account credentials and check the "I am an agent" checkbox to log in as an agent. Have fun exploring the features as an agent!

<br>

Features include:
- Viewing list of customers that are assigned to the agent
- Viewing assigned customer's polic(ies)

[Back To The Top](#Insurance-Aggregator)

<br>

## Future Work
Looking to explore and implement authentication and rate limiting plugins on the API gateway. This is of interest because access control and prevention of malicious traffic are one of the key benefits of an API gateway. 

[Back To The Top](#Insurance-Aggregator)
