<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# APIs with Lambda + API Gateway

**Project Link:** [View Project](https://nextwork.ai/projects/2af91800-3f42-5fc1-b405-c5069180c0df)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/2af91800-3f42-5fc1-b405-c5069180c0df_c9d0e1f2)

## Introducing Today's Project!

In this project, I will demonstrate how to build a serverless API using AWS Lambda and Amazon API Gateway. I'm doing this project to learn how to set up serverless backend infrastructure, create and deploy Lambda functions, configure API resources and methods, and connect them together as part of a three-tier architecture.

### Tools and concepts

Services I used were Amazon API Gateway and AWS Lambda. Key concepts I learnt include Lambda functions, REST APIs, Lambda proxy integration, API stages, and API documentation.

### Project reflection

This project took me approximately 20 minutes to complete! The most challenging part was making sure the Lambda proxy integration and query parameters were correctly mapped so the function could successfully parse the incoming request. It was most rewarding to deploy the API to a live stage and see the public invoke URL successfully connect my front door API gateway to backend logic.

I chose to do this project today because I wanted to gain hands-on experience building serverless backend architectures with AWS Lambda and Amazon API Gateway as part of the three-tier series. Something that would make learning with NextWork even better is having more interactive troubleshooting sandboxes or step-by-step architecture diagrams that visually map out how payloads flow between services in real time!

## Lambda functions

AWS Lambda is a serverless compute service that lets you run code without managing servers. I'm using Lambda in this project to build the backend logic tier that processes incoming requests and retrieves user records from a database.

The code I added to my function will process incoming requests by extracting the user ID from query string parameters, querying an Amazon DynamoDB table to retrieve the matching user record, and returning the result with a 200 status code or handling errors if the data cannot be found or retrieved.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/2af91800-3f42-5fc1-b405-c5069180c0df_a1b2c3d5)

## API Gateway

APIs are interfaces that allow different software systems to communicate with each other. There are different types of APIs, like REST, HTTP, and WebSocket APIs, which are suited for different use cases like web applications or real-time chat. My API is a REST API configured with Amazon API Gateway to connect users with our backend Lambda function.

Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. I'm using API Gateway in this project to act as the front door for our Lambda function, managing and routing incoming HTTP requests to securely connect users with our backend logic tier.

When a user makes a request to the API, API Gateway receives the request, acts as the front door, and forwards it directly to the AWS Lambda function. Lambda then processes the request, interacts with backend services like DynamoDB, and sends the response back through API Gateway to the user.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/2af91800-3f42-5fc1-b405-c5069180c0df_m3n4o5p6)

## API Resources and Methods

An API is made up of resources, which are individual endpoints that represent different parts of your application and handle specific operations or data paths.

Each resource consists of methods, which are the actions you can perform on that resource using standard HTTP commands like GET to retrieve data, POST to add data, PUT to update data, and DELETE to remove data.

I created a GET method for the users resource and configured it with Lambda proxy integration to connect incoming HTTP requests directly to our RetrieveUserData Lambda function so it can process requests and return the requested data.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/2af91800-3f42-5fc1-b405-c5069180c0df_c9d0e1f2)

## API Deployment

When you deploy an API, you deploy it to a specific stage. A stage is a snapshot of your API at a specific point in time that controls its version and availability for live traffic. I deployed to a new stage named prod so that my API becomes accessible through a public invoke URL for production traffic.

To visit my API, I copied the invoke URL from the prod stage in API Gateway and accessed it in a new browser tab. The API displayed an error because we haven't set up our DynamoDB table yet.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/2af91800-3f42-5fc1-b405-c5069180c0df_3ethryj2)

## API Documentation

For my project's extension, I am writing API documentation because I want to provide a detailed, user-friendly description of my API's functionality, endpoints, and base URL so other developers can easily understand and use it. You can do this in the API Gateway console by navigating to the Documentation section, creating documentation parts with descriptions and base URLs, and publishing them to your API stage.

Once I prepared my documentation, I can publish it to the prod stage. You have to publish your API to a specific stage because it ensures your documentation aligns with the correct API version and associates the snapshot with the live environment where it controls availability for users.

My published and downloaded documentation showed me automatically generated metadata like the API's title, version, and the /users resource with its GET method, alongside the custom JSON documentation I added for my base URL and description.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/2af91800-3f42-5fc1-b405-c5069180c0df_z9a0b1c2)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/2af91800-3f42-5fc1-b405-c5069180c0df)*
