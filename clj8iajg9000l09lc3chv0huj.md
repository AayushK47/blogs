---
title: "Production-Grade Workflow for File Uploads"
seoTitle: "Production-Grade Workflow for File Uploads"
seoDescription: "This blog discusses how we can upload and manage a file in your application."
datePublished: Fri Jun 23 2023 11:46:04 GMT+0000 (Coordinated Universal Time)
cuid: clj8iajg9000l09lc3chv0huj
slug: production-grade-workflow-for-file-uploads
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687520549327/a33d8f22-d436-4331-8e72-274491ac29f7.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1687520635197/a7828764-152c-4b04-b5eb-2a7d0557242d.png
tags: cloud, aws, frontend, s3, file-upload

---

Let's say you want to create a file upload feature on your app. What is the best way to design it? If you're a web developer, you know that uploading files is a common task that can be trickier than it seems. There are many factors to consider, including security, performance, and scalability. Let's discuss how we can upload and manage a file in your application.

## Sending the file to the backend

One way to manage this feature is by sending the data to the backend through a request. That is half the problem solved. Once the file reaches the backend, we need to decide how we are going to store this file, such that it can be used easily next time. So again, we can go three ways here:-

* Save the file as a blob in the database.
    
* Save the file in the server, and save the file path in the database for later use.
    
* Send the file to a cloud-storage service, and save its URL in the database.
    

Let's discuss each of these.

The first approach is probably the easiest. One query and we are done. But it is the worst approach. The reason is that files can be extremely large. If we save them in the database, not only will it consume a lot of our resources, but it will also slow down our query execution as fetching blob type is generally slower.

The second approach is better. You can easily save the file on your server. Its file path, which is a string will not cause any issue in the database and we can easily save it in the database. But again, a server does not have infinite storage capacity. At some point, we will have to increase the storage.

Out of the three, the third approach is the best. A cloud-storage service is generally highly available, scalable, fully managed, secure and works on a pay-as-you-go model. So instead of storing the file on your server or database, why not simply store it on the cloud, and save a link to that file on your database? The permissions can also be managed this way, so anyone without sufficient permissions won't be able to access the file.

But all these approaches still have an issue. Sending files from the front end to the back end may slow down depending on file size, network bandwidth, etc.

## Uploading files directly from the frontend

Uploading files from the front end seems like a good idea. But how do we do it?

It's pretty easy. All you need is an upload URL of the S3 object. But remember the URL must be secure. It should have limited permissions and those permissions should expire after some time. So a secure signed URL has to be sent from the back end to the front end, which it will use to upload the file. In response to the file upload request, the front end will receive a slug, which they can send to the backend to be stored in the database.

Now, I think that what may be bugging you would be - "How do we access the file from the slug?". Well, we can get a signed URL with read permissions from the slug of the file object. When a user requests the file, we can get a signed URL from the slug and then return it from the backend.

## Let's Just Do It

Let's try and implement this idea. For this tutorial, I will be using AWS S3. To follow along, you need a free AWS account and AWS CLI installed on your machine. The code will be written in JavaScript and React, but the logic is the same in every language. If you understand the logic well, you will be able to implement it in your favourite language.

### AWS Configurations

Assuming you have an AWS account and you have installed the AWS CLI, the first thing we need to do is create the S3 buckets with appropriate permissions. Now, to do this, you can use the AWS management console, but for this demo, I will be using the AWS CLI.

To configure the AWS CLI you need an access key and a secret key, which you can get from the IAM console on your AWS account by following these steps:-

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684486464918/bb188326-bba1-41dc-93b9-a1996ee9887b.png align="center")

Once you have the keys, fire up your terminal and run:-

```bash
aws configure
```

The *AWS configure* command will ask for the access key, secret key and AWS region. Once you have entered all these details, you are all set to create the S3 buckets.

### Creating S3 Buckets

Creating a bucket is easy enough. Just run the following command on your terminal:-

```bash
aws s3api create-bucket --bucket <bucket-name> --region <regoin name>
```

We're not done yet, we need to configure CORS for this bucket. To do that, run the following:-

```bash
aws s3api put-bucket-cors --bucket <your-bucket-name> --cors-configuration '{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "HEAD"],
      "AllowedOrigins": ["*"],
      "ExposeHeaders": []
    }
  ]
}'
```

Here I have allowed all origins and headers, along with `GET` and `HEAD` methods. You can change this configuration as per your requirement.

### File Uploader Component

Let's start with the UI:-

```javascript
function App() {
    return (
        <div>
            <form action="">
                <input type="file" />
            </form>
        </div>
    )
}

export default App;
```

It's a simple react component that renders a file input. I have used no styles here, just to keep it simple. Let's add some functionality to it.

When we click on the upload button, we get a popup to select a file, and when we select the file, the input value changes. So to override the default behaviour with our functionality, we need to use the onChange prop on input:-

```javascript
function App() {
    handleFileUpload(event) {
        // API call to get signed url for uploading the file
        let response = await fetch.get('<INSERT API ENDPOINT>')
        response = await response.json()
        const uploadUrl = response.data.url;
        // API to call upload image to S3
        const uploadUrl = await fetch.put(uploadUrl, event);
        if(uploadUrl.status === 200) {
            // Call the api to save the slug in your database
        }
    }
    return (
        <div>
            <form action="">
                <input type="file" />
            </form>
        </div>
    )
}

export default App;
```

Here, I have added `handleFileUpload` method which uploads the file on S3. First, we send a request to our backend, which returns a signed URL through which we can upload the file. Next, we send a put request to the signed URL with the file object as the body. This will upload the file to the S3 bucket, and we get the s3 URL and slug in response. Next, you can save the slug in your database if you want to use it later. Your backend can generate a signed URL using the slug with just read permissions.

But, we are missing something here, aren't we? Oh yeah, we need to create endpoints which return signed URLs. Let's create them in the next section.

### Generating Signed URLs

Depending on which framework you use, the boilerplate of your project may change. All frameworks will have a service with common logic. Let's see how this service looks:-

```javascript
const AWS = require('aws-sdk');
const { v4 } = require('uuid');

const s3 = new AWS.S3({
    accessKeyId: accessKeyId,
    secretAccessKey: secretAccessKey,
    region: region
});

function generateSignedS3Url() {
  const expires = new Date();
  expires.setMinutes(expires.getMinutes() + 10);

  // Generate the signed URL parameters
  const params = {
    Bucket: bucketName,
    Key: v4(),
    Expires: expires
  };
  const signedUrl = s3.getSignedUrl('putObject', params);

  return signedUrl;
}
```

In the above snippet, we created an S3 object from the `aws-sdk`, and then we defined the `generateSignedS3Url` function. To generate a signed URL, we need to provide the bucket where we want to store the file, a key, which is basically the file name, and expires, which is the time when the permission on our signed URL expires. `putObject` is an AWS S3 action which allows put request on an object. We use `getSignedUrl` on the s3 object to generate the signed URL, and return this signed URL.

Now, this function can be called on your controller to get the signed URL and return as a response.

## Summary

In conclusion, when designing a file upload feature, consider security, performance, and scalability storing files as blobs in the database is convenient but resource-intensive. Saving file paths on the server is efficient but limited by storage capacity. The best approach is using a cloud-storage service like AWS S3, offering availability, scalability, and security and would involve uploading files directly from the front end using secure signed URLs.

Thank you for reading, we value your feedback. Happy coding!