# amazon-rekognition-example

Facial detect a list of customers, based on their facial picture, using the
Amazon [Rekognition](https://aws.amazon.com/rekognition/) and [Lambda](https://aws.amazon.com/lambda/)
web services along with [S3](https://aws.amazon.com/s3/) and [DynamoDB](https://aws.amazon.com/dynamodb/) databases.

> ### Watch the [video tutorial](https://youtu.be/oHSesteFK5c)

## Installation

1. Install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

1. Configure
   ```
   aws configure
   ```

1. Create A Collection On AWS Rekognition
   ```
   aws rekognition create-collection --collection-id facerecognition_collection --region us-east-1
   ```

1. Create Table On DynamoDB
   ```
   aws dynamodb create-table --table-name facerecognition \
   --attribute-definitions AttributeName=RekognitionId,AttributeType=S \
   --key-schema AttributeName=RekognitionId,KeyType=HASH \
   --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
   --region us-east-1
   ```

1. Create S3 Bucket
   ```
   aws s3 mb s3://bucket-name --region us-east-1
   ```

## FAQ

### Q: What is Amazon DynamoDB?

Amazon DynamoDB is a database located at Amazon AWS.
We define it to contain a list of Customers that have two properties:
- {string} `Faceprint` as a primary-key.
- {string} `Name`.

```mermaid
erDiagram
    CUSTOMER {
        string Faceprint PK
        string Name
    }
```

### Q: What is a "Faceprint"?

A `Faceprint`, also known as `faceID`, is some-kind of a string key, that is
generated as a response of analysing a specific image that was posted to
Amazon Rekognition (See [graph](https://github.com/taljacob2/amazon-rekognition-example/tree/feat-update-readme#insert-a-new-image-to-dynamodb)).
On the other way, we can also send an already-generated `Faceprint` to
Amazon Rekognition and it would respond us with the most accurate `Faceprint`
it detects that had been already stored on the DynamoDB (See [graph](https://github.com/taljacob2/amazon-rekognition-example/tree/feat-update-readme#detect-name-from-a-test-image)).

---

## Logic Explanation

### Insert A New Image To DynamoDB

```mermaid
sequenceDiagram
    autonumber
    
    actor U as User
    note right of U: Insert a new image to DynamoDB
        
    participant S3 as S3 Bucket
    participant LambdaFunction as Lambda Function
    participant DynamoDB
    
    U->>S3: insert a `.jpg` to
    S3->>LambdaFunction: triggers
    LambdaFunction->>DynamoDB: extracts Name and Faceprint from the image, and inserts to
    
```

### Detect Name From A Test Image

```mermaid
sequenceDiagram
    autonumber
    
    actor U as User
    note right of U: Detect Name from a test image
    
    participant Rekognize
    participant DynamoDB    
    
    U->>Rekognize: upload a test image
    Rekognize->>Rekognize: generate Faceprint from image
    Rekognize->>DynamoDB: send Faceprint
    DynamoDB->>DynamoDB: is Faceprint in DB?
    alt Faceprint found
        DynamoDB->>DynamoDB: find the corresponding Name for that Faceprint
        DynamoDB-->>U: response with Name (OK)
    else Faceprint not found
        DynamoDB-->>U: exit (ERROR)
    end
```
