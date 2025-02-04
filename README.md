# NCAA Game Highlights Project
This project processes and manages NCAA game highlights using advanced workflows including AWS services, Docker, and structured folder organization. The repository enables efficient handling of raw and processed game videos on S3, and provides JSON-based metadata storage for game data. Below, you'll find instructions and details about this project, including used commands, folder structure, and an overview of the architecture.
## **Project Overview**
- **API Integration:** The project uses a Sport Highlights API to fetch game highlight data.
- **Data Storage:** Videos and metadata are stored and managed using AWS S3.
- **Video Processing:** Raw videos are processed using a containerized application, which is executed with Docker containers.
- **Cloud Services**: Integration with AWS Secrets Manager and AWS Elemental MediaConvert ensures secure and scalable processes for handling sensitive keys and media processing workflows.

## **Prerequisites**
Before running the project, ensure you have the following:
1. **AWS CLI** configured on your machine.
2. **Docker** installed and running.
3. An `.env` file containing environment variables required by the containerized application (this is mentioned in `.gitignore` and therefore not tracked by Git).

## **Setup and Deployment Instructions**
### **1. Store the API Key Securely**
To store the Sport Highlights API key securely in AWS Secrets Manager, run the following command:
``` shell
aws secretsmanager create-secret \
    --name my-api-key \
    --description "API key for accessing the Sport Highlights API" \
    --secret-string '{"api_key":<your_key>}' \
    --region us-east-1
```
This command creates a secret named **`my-api-key`** in the **us-east-1 region** and stores the API key securely.
### **2. Create an IAM Role**
- In the console search bar type "IAM"

- Click Roles -> Create Role

- For the Use Case enter "S3" and click next

- Under Add Permission search for AmazonS3FullAccess, MediaConvertFullAccess and AmazonEC2ContainerRegistryFullAccess and click next

- Under Role Details, enter "HighlightProcessorRole" as the name

- Select Create Role

- Replace the attached "Trust Relationship" with the json object in "S3_IAM_Role file".

### **3. Create an S3 Bucket**
Create an S3 bucket to store videos and related JSON data:
``` shell
aws s3api create-bucket --bucket <bucket_name> --region <region_name>
```
This bucket will be used for:
- Storing JSON data for game highlights.
- Hosting raw unprocessed videos.
- Storing processed videos after they are processed.

### **4. Build the Docker Image**
A Docker container is used to handle the processing of video files. To build this image, use the Dockerfile provided in the repository with the following command:
``` shell
docker build -t highlight-processor .
```
Ensure the Docker daemon is running on your system before proceeding, because I use windows I have to open the docker desktop app to ensure i don't get this error (PS: I always forget and received the "daemon not running" error before i open the docker app, don't be like me hehe!)
### **5. Run the Docker Container**
Execute the following command to start the container with the necessary environment variables (specified in your `.env` file):
``` shell
docker run --env-file .env highlight-processor
```
The container will process the raw videos from the `videos` folder and upload the processed videos to the designated folder in the S3 bucket.
### **6. Media Processing Workflow**
Once the videos are processed successfully, you should see a "Job Summary" in **AWS Elemental MediaConvert** indicating the completed task. The processed videos will be available in the following folder structure in the S3 bucket:

#### **Folder Structure**
- **`ncaahighlights`**: Stores JSON data about game highlights metadata.
- **`videos`**: Contains the raw unprocessed videos.
- **`processed_videos`**: Stores the videos after they have been processed.

### Components:
1. **API Key Management**:
    - AWS Secrets Manager securely stores sensitive API keys.

2. **Raw Video Collection**:
    - Videos are uploaded to the `videos` folder (stored in the S3 bucket).

3. **Processing**:
    - Docker container executes the processing workflow.
    - Resulting videos are uploaded to `processed_videos` folder in the same bucket.

4. **Output**:
    - AWS MediaConvert's processing summary is accessible in the AWS Console.

## **How It Works**
1. The raw game videos are stored in the designated folder (`videos`).
2. The Docker container processes these videos, using configurations defined in the `.env` file.
3. Processed videos are uploaded to an S3 bucket for further use.
4. Metadata about the highlights is stored in JSON format within the `ncaahighlights` folder in the same bucket.

## **Additional Notes**
- Ensure all relevant permissions are configured within AWS IAM to use Secrets Manager, S3, and MediaConvert.
- Update the `.env` file as required for any environment-specific configurations.
