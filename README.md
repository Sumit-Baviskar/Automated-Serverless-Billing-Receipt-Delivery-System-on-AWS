# **Automated Serverless Billing Receipt Delivery System on AWS**


## **💡 Problem Statement** :
Manually sending monthly billing receipts to customers is time-consuming and error-prone. This system solves that problem by automating the entire process end-to-end, ensuring each customer receives an accurate, personalized, and professional receipt every month.



## **🚀 Introduction :**
In today’s fast-paced digital world, delivering a seamless and secure customer experience is more important than ever. As companies scale, tasks like manually sending monthly billing receipts to hundreds (or even thousands) of customers quickly become unmanageable, error-prone, and inefficient.

To address this challenge, I designed and implemented an Automated Serverless Billing Receipt Delivery System on AWS — a fully automated solution that eliminates manual intervention and ensures every customer receives accurate, timely, and professional billing receipts each month.

This project leverages the power of AWS serverless services — including S3, Lambda, SQS, and SES — to create a fully event-driven, scalable architecture. By uploading a simple CSV file to an S3 bucket, the system automatically parses and processes each customer’s billing data, calculates total charges, formats a detailed plain-text receipt, and securely sends it to the customer’s email address.



## **🏗️ System Architecture :**

The **system is entirely serverless and relies on the following AWS services** :

 - **Amazon S3** : Stores uploaded CSV files containing customer billing data.

 - **AWS Lambda (Parser Function)** : Parses the CSV, groups services by customer, and sends messages to SQS.

 - **Amazon SQS** : Decouples data processing and ensures reliable delivery of customer-specific data.

 - **AWS Lambda (Worker Function)** : Picks up customer data from SQS, formats a personalized email body, and sends it.

 - **Amazon SES (Simple Email Service)** : Sends billing emails to each customer.

This architecture allowed me to build a scalable and cost-efficient pipeline that can handle varying workloads without worrying about server management.


## **Project video link :**

 [Video Link](https://drive.google.com/file/d/10mgVvskvuKXcH5DYC9TvRtCR0faQCcrE/view?usp=sharing)



## **📨 Key Features :**

 - Fully automated billing receipt emails.

 - Serverless design (no servers to manage).

 - Decoupled and scalable architecture.

 - Clean, formatted plain-text email receipts.

 - Easily extendable (e.g., adding PDF attachments, multi-language support).


## **🛠️ Tech Stack :**

 - **AWS Lambda**

 - **AWS S3**

 - **AWS SQS**

 - **AWS SES**

 - **Python (boto3)**

## **🚨 How It Works :**

 1️⃣ Upload a CSV file to the S3 bucket.
 
 
 2️⃣ Lambda 1 is triggered → parses CSV → groups data → pushes messages to SQS.

 
 3️⃣ Lambda 2 (triggered by SQS) → generates email body → sends email via SES to each customer.


## ✅ **Dummy CSV data file**
    customer_id,customer_name,email,service_name,charges
    CUST001,John Doe,john@example.com,Cloud Backup,50
    CUST001,John Doe,john@example.com,API Hosting,30
    CUST002,Jane Smith,jane@example.com,Monitoring,20




## ✅ ✅ **Step-by-step setup :**

### 1️⃣ Create your S3 bucket

 📍 **Steps :**

   - Go to AWS Console → S3 → Create bucket.(Example bucket name: company-billing-bucket)

### 2️⃣ Create your SQS queue

 📍 **Steps :**

   - Go to AWS Console → SQS → Create queue.( Example queue name: billing-customer-queue.) → Create

   - Copy the Queue URL

### 3️⃣ Create your Lambda function (Example name Parser lambda)

📍 **Steps :**

   - Go to AWS Console → Lambda → Create function.( Example Function name: parser-customer-lambda.)

   - Click Create function.

### 4️⃣ Add environment variable for SQS URL

📍 **Steps :**

   - In Lambda console → Configuration → Environment variables → Edit → Add.

   - Key: SQS_URL.

### 5️⃣ **Add Lambda code :**
 

    import boto3
    import csv
    import json
    import os
    import urllib.parse

    # Create AWS clients
    s3 = boto3.client("s3")
    sqs = boto3.client("sqs")

    # Get SQS queue URL from environment variable
    SQS_URL = os.environ["SQS_URL"]

    def lambda_handler(event, context):
        # Log the received event for debugging
        print("✅ Event received:", json.dumps(event))
    
        # Get the first record (in case multiple records are uploaded at once)
        record = event["Records"][0]
        bucket = record["s3"]["bucket"]["name"]
        key = urllib.parse.unquote_plus(record["s3"]["object"]["key"])
    
        print(f"🪣 Bucket: {bucket}, 📄 Key: {key}")

        # Get the object (CSV file) from S3
        response = s3.get_object(Bucket=bucket, Key=key)
        csv_content = response["Body"].read().decode("utf-8").splitlines()
        reader = csv.DictReader(csv_content)

        # Dictionary to store customer-wise data
        customer_data = {}

        # Iterate over each row in the CSV
        for row in reader:
            cust_id = row["customer_id"]
            cust_name = row["customer_name"]
            email = row["email"]
            service_name = row["service_name"]
            charges = float(row["charges"])

            # Initialize data for new customer if not already present
            if cust_id not in customer_data:
                customer_data[cust_id] = {
                    "customer_id": cust_id,
                    "customer_name": cust_name,
                    "email": email,
                    "services": []
                }

            # Append service details for the customer
            customer_data[cust_id]["services"].append({
                "service_name": service_name,
                "charges": charges
            })

        # Send each customer's data as a separate message to SQS
        for cust_id, data in customer_data.items():
            sqs.send_message(
                QueueUrl=SQS_URL,
                MessageBody=json.dumps(data)
            )
            print(f"📤 Message pushed to SQS for customer: {cust_id}")

        return {
            "statusCode": 200,
            "body": "✅ Messages pushed to SQS for each customer"
        }



### 6️⃣ Set Lambda permissions
  
 📍 **Steps :**
 
  - **IAM permissions :**
    
     - Go to Lambda console → Configuration → Permissions → Execution role.

     - Click Edit → Attach policies.

         - Attach:

             - AmazonS3ReadOnlyAccess

             - AmazonSQSFullAccess



### 7️⃣ **Add S3 trigger to Lambda**

📍 **Steps :**

   - Go to S3 → Your bucket → Properties → Event notifications → Create event notification.

   - Name: billing-upload-trigger.

   - Event type: PUT (object created).

   - Prefix (optional): e.g., billing-files/.

   - Suffix (optional): .csv.

   - Destination: Lambda function, choose your Lambda.


### 8️⃣ **Configure SES (Simple Email Service) :**

📍 **Steps :**
    
   - Go to SES console → Verify email address.

   - Verify your "from" email address (e.g., billing@yourcompany.com).

   - If in sandbox mode, also verify "to" email addresses (test customers).

   - Check that verification status shows verified.


### 9️⃣ **Create your Lambda function (Example name Parser lambda)**

📍 **Steps :**

   - Go to AWS Console → Lambda → Create function.( Example Function name: worker-customer-lambda.)

   - Click Create function.



### 1️⃣0️⃣ **Add environment variable for SQS URL**

 📍 **Steps :**
 
   - In Lambda console → Configuration → Environment variables → Edit → Add.

   - Key: SQS_URL.

    import boto3
    import json
    import os
    
    # Create AWS SES client
    ses = boto3.client("ses")

    # Get sender email from environment variable
    SENDER_EMAIL = os.environ["SENDER_EMAIL"]

    def lambda_handler(event, context):
        try:
            # Process each SQS message
            for record in event["Records"]:
                # Parse JSON message body
                message = json.loads(record["body"])
                customer_name = message["customer_name"]
                email = message["email"]
                services = message["services"]
    
                # Calculate total charges for this customer
                total_charges = sum(service["charges"] for service in services)

                # Build plain-text email body
                body_text = f"""Hi {customer_name},
    
    Thank you for choosing [Your Company Name]!
    Here’s a summary of the services you used in Last Month:

    --------------------------------------------------------
    Service Name         | Amount (USD)
    --------------------------------------------------------
    """

                # Add each service as a row in the email
                for svc in services:
                    # Left-align service name in 20 chars, right-align amount
                    body_text += f"{svc['service_name']:<20} | ${svc['charges']:.2f}\n"

                # Add total and closing remarks
                body_text += f"""--------------------------------------------------------
    Total Charges        | ${total_charges:.2f}
    --------------------------------------------------------

    Your support enables us to keep providing you with top-notch services.
    If you have any questions about your bill or need assistance, feel free to reply to this email or contact our support team.

    Thank you for your continued trust!

    Best regards,
    [Your Company Name] Billing Team
    xxxxxxxxx@company.com | xxxxxxhelp@support.com
    """

                # Send email using AWS SES
                ses.send_email(
                    Source=SENDER_EMAIL,
                    Destination={"ToAddresses": [email]},
                    Message={
                        "Subject": {"Data": "Your Monthly Service Receipt - July 2025"},
                        "Body": {
                            "Text": {"Data": body_text}
                        }
                    }
                )

                # Log success for this customer
                print(f"✅ Email sent successfully to: {customer_name} ({email})")

            # Return success response after processing all messages
            return {
                "statusCode": 200,
                "body": "✅ All SQS messages processed and emails sent successfully."
            }

        except Exception as e:
            # Log any errors
            print(f"❌ Error: {str(e)}")
            return {
                "statusCode": 500,
                "body": f"❌ Error processing SQS messages: {str(e)}"
            }



### 1️⃣1️⃣ **Attach environment variables :**

 📍 **Steps :**
 
   - In Lambda console → Configuration → Environment variables → Edit → Add.

         Key: SENDER_EMAIL
         Value: billing@yourcompany.com



### 1️⃣2️⃣ **Attach IAM permissions**

 📍 **Steps :**
   
   - Go to Lambda console → Configuration → Permissions → Execution role.

   - Click Edit → Attach policies.

     - Attach:

        - AmazonS3FullAccess

        - AmazonSESFullAccess

        - AmazonSQSFullAccess (or ReceiveMessage & DeleteMessage)


### 1️⃣3️⃣ **Set SQS as trigger for Worker Lambda**
 
 📍 **Steps :**
   
   - Go to Lambda console → Configuration → Triggers → Add trigger → SQS.

   - Select your queue (e.g., billing-customer-queue).

   - Configure batch size (default is 10 — can set to 1 for simpler testing).


### 1️⃣4️⃣ **Test end-to-end**

 📍 **Steps :**
 
   - Upload CSV to S3 (Parser Lambda is triggered).

   - Parser Lambda pushes messages to SQS.

   - Worker Lambda gets SQS messages, generates PDFs, uploads to S3, and sends email via SES.

## **💡 Sample Output Email (Email Received by Customer)** :

<img width="1105" height="608" alt="Image" src="https://github.com/user-attachments/assets/aa98cd45-42ca-451f-864a-e7247f3ebace" />


## ⚡ **Key benefits :**

  - 🚫 Eliminates manual effort: Fully automates billing receipt generation and delivery.

  - 💬 Improves customer experience: Sends clear, professional, personalized receipts every month.

  - 🔒 Ensures data privacy and security: Implements robust measures to safeguard sensitive billing data.

  - ☁️ Scales effortlessly: Serverless architecture handles any number of customers without infrastructure management.


