# APIs with Lambda + API Gateway — Cheat Sheet

Lesson notes for reviewing later. The project write-up is in [README.md](README.md), and the exported API documentation is in [project-files/](project-files/).

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [AWS Lambda](#aws-lambda)
4. [Runtime and architecture](#runtime-and-architecture)
5. [APIs and API Gateway](#apis-and-api-gateway)
6. [API types and endpoint types](#api-types-and-endpoint-types)
7. [Resources and methods](#resources-and-methods)
8. [Integration types and Lambda proxy integration](#integration-types-and-lambda-proxy-integration)
9. [Stages and the invoke URL](#stages-and-the-invoke-url)
10. [API documentation: Swagger and OpenAPI](#api-documentation-swagger-and-openapi)
11. [Build order and cleanup](#build-order-and-cleanup)
12. [Self-quiz](#self-quiz)

---

## What I built

- ⚡ **A Lambda function** (`RetrieveUserData`, Node.js runtime) that reads a `userId` from the request's query string, looks the user up in a DynamoDB table and returns the result.
- 🚪 **A REST API in API Gateway** (`UserRequestAPI`, Regional endpoint) with a `/users` resource and a **GET** method, connected to the function using **Lambda proxy integration**.
- 🚀 **A `prod` stage**, giving the API a public **invoke URL**.
- 📄 **API documentation**, published to `prod` and exported as a Swagger file.

This is the **logic tier** (the backend that processes requests) of a three-tier architecture. The CloudFront project built the presentation tier, and the DynamoDB table (the data tier) comes next.

```
  User's browser
        │  GET https://xxxx.execute-api.<region>.amazonaws.com/prod/users?userId=1
        ▼
  ┌──────────────────────┐   proxy integration  ┌──────────────────────┐        ┌──────────────────┐
  │ API Gateway          │ ───────────────────► │ Lambda function      │ ─────► │ DynamoDB table   │
  │ REST API, prod stage │   whole request      │ RetrieveUserData     │        │ (next project)   │
  │ /users → GET         │ ◄─────────────────── │ returns status+body  │ ◄───── │                  │
  └──────────────────────┘                      └──────────────────────┘        └──────────────────┘
        the "front door"                              the logic tier                 the data tier
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **AWS Lambda** | Runs your code without you managing servers; you pay only while it runs | A taxi: you pay per ride, not for owning a car |
| **Lambda function** | A piece of code you run in Lambda | One taxi trip doing one job |
| **Runtime** | The environment your code runs in (e.g. Node.js, Python) | The kitchen a recipe is cooked in |
| **Node.js** | A runtime that lets JavaScript run on a server, not just in the browser | Taking a home cook's recipes into a restaurant kitchen |
| **Architecture** | The processor type your function runs on (e.g. x86_64) | The engine type under the taxi's hood |
| **API** | A way for software systems to talk to each other | A messenger carrying requests and replies |
| **API Gateway** | AWS service to create, publish, monitor and secure APIs | The front door with a receptionist |
| **REST API** | An API that uses HTTP methods to act on resources | A menu where each dish has a standard order form |
| **Endpoint type** | How widely the API is available (Regional, Edge-Optimized, Private) | A local shop, a global chain, or a staff-only canteen |
| **Resource** | An individual endpoint/path in the API, like `/users` | A department in a store |
| **Method** | An action on a resource: GET, POST, PUT, DELETE | What you ask the department to do |
| **Integration type** | The backend that fulfils the request (here, Lambda) | Which back office the receptionist sends you to |
| **Lambda proxy integration** | API Gateway passes the whole request to Lambda untouched | Forwarding the sealed envelope instead of retyping the letter |
| **Stage** | A named, deployed version of the API (e.g. `dev`, `test`, `prod`) | Rehearsal vs opening night |
| **Invoke URL** | The URL where a stage of your API can be called | The shop's street address |
| **Swagger / OpenAPI** | Standard formats for describing a REST API | A standard blueprint layout every builder can read |

## AWS Lambda

- **Serverless:** you supply the code; AWS manages the servers.
- **Runs only when needed**, so you don't pay for idle time.
- **Scales automatically**, from a few requests a day to thousands per second.
- A **Lambda function** is one piece of code: processing data, responding to events or running automated tasks.
- The **function's page** in the console shows its code, configuration (memory, timeout) and performance metrics. That's where you paste and deploy the code.

## Runtime and architecture

| Setting | What it is | In this project |
|---|---|---|
| **Runtime** | The environment that runs your code and has the dependencies it needs. Python code needs a Python runtime, and so on | **Node.js** |
| **Architecture** | The processor type the function runs on | **x86_64** (widely used) |

- **Node.js** takes JavaScript to the **server side**. Normally JavaScript runs in the browser after the page loads (animations, form checks). With Node.js it runs on a server *before* anything reaches the browser, so one language works for both front end and backend.
- **"Serverless, but hardware?"** Serverless means *you* don't manage servers. Your code still runs on physical servers behind the scenes, so you still pick a processor architecture, and AWS manages it for you.

## APIs and API Gateway

- An **API** carries requests and responses between systems. Here it carries requests from the user's browser to the Lambda function.
- **API Gateway** manages incoming traffic, sends it to the right service and makes sure only authorized requests get through.
- **The relationship:** API Gateway is the **front door**. It receives the request and forwards it to Lambda. Lambda processes it and sends the response back through API Gateway to the user.

**Why not expose Lambda directly?** It isn't best practice. Lambda on its own lacks API management features. API Gateway adds:
- **authentication and authorization**, and
- **API management**, such as request routing.

## API types and endpoint types

| API type | Best for |
|---|---|
| **REST** | A standard web API with the full feature set ✅ *this project* |
| **HTTP** | A simpler, cheaper, lower-latency API that mostly just routes requests |
| **WebSocket** | Real-time, two-way communication (e.g. chat) |

| Endpoint type | Available | Good for |
|---|---|---|
| **Regional** | In one AWS Region; low latency for clients there | Localized apps ✅ *this project* |
| **Edge-Optimized** | Through CloudFront edge locations worldwide | Global apps |
| **Private** | Only from inside your VPC | Internal networks |

> **Link back:** Edge-Optimized endpoints use the same CloudFront edge network as the previous project.

The **API's page** in the console is the central hub: its resources, methods, deployments (stages) and settings.

## Resources and methods

- **Resources** are the individual endpoints that handle different parts of the API. A messaging app might have one for messages and one for user profiles.
- **Resource path** = the URL path, e.g. `/users`. **Resource name** = what the console calls it, so you can manage it easily.
- **Methods** are the actions on a resource, based on standard HTTP methods:

| Method | Does | Example |
|---|---|---|
| **GET** | Retrieve data | Get a user's profile ✅ *this project* |
| **POST** | Add data | Create a new user |
| **PUT** | Update data | Change a user's email |
| **DELETE** | Remove data | Delete a user |

## Integration types and Lambda proxy integration

The **integration type** is the backend that fulfils the request, and decides how API Gateway passes the request to it and handles the response. With **Lambda** as the integration:

1. API Gateway gets a request and sends it to the Lambda function.
2. The function processes it and sends the result back to API Gateway.
3. API Gateway passes the response on to the user.

The **function you select** is the one that runs when that method is called: GET on `/users` triggers `RetrieveUserData`.

**Lambda proxy integration** (Extra for Experts):

| | Without proxy integration | With proxy integration |
|---|---|---|
| The request | API Gateway breaks it down and reformats it for Lambda | API Gateway passes the **whole** request (headers, query parameters, path parameters, body) straight to Lambda |
| The response | API Gateway maps it back into the format the client expects | Lambda must return it in the expected shape itself |
| Who does the work | API Gateway (mapping templates) | The Lambda function |

> **Tip:** with proxy integration, the function must return an object shaped like `{ statusCode, headers, body }`, with `body` as a **string** (e.g. `JSON.stringify(data)`). Otherwise API Gateway returns `502 Bad Gateway`. The query parameter arrives as `event.queryStringParameters.userId`.

## Stages and the invoke URL

- You **deploy** an API to a **stage**. Developers usually build in a **dev** stage, test in a **test** stage, then release to **prod**. That way you control who gets which version and when.
- The **invoke URL** is where the stage can be called: `https://<api-id>.execute-api.<region>.amazonaws.com/prod`. Live apps put the **prod** invoke URL in their code.
- **First visit result:** an error, because there's no DynamoDB table yet. The front door and logic tier are connected, but the data tier doesn't exist.

> **Note:** the lesson calls a stage "a snapshot of your API" (and the heading says "stage in Lambda", but stages belong to **API Gateway**). Strictly, the **deployment** is the snapshot, and a **stage** is a named pointer to a deployment. Redeploying to `prod` moves the pointer to a new snapshot. Changes you make in the console don't go live until you **deploy** again.

## API documentation: Swagger and OpenAPI

- **API documentation** describes the API's endpoints (`/users`), methods (GET), parameters (`userId`) and responses (success or errors), so other developers can use it correctly.
- **Documentation types** set the scope: the **API** as a whole, or individual **resources**, **methods** and **models**. This project documented the whole API with a `description` and `baseURL`.
- Documentation must be **published to a stage** (here, `prod`) so it matches that version of the API.

| Spec | What it is |
|---|---|
| **Swagger 2.0** | A popular format for describing REST APIs: endpoints, operations, parameters, authentication, contact info. Readable by people and machines |
| **OpenAPI 3** | The newer version of Swagger. Swagger was donated to the OpenAPI Initiative, so "Swagger 3.0" became **OpenAPI 3** |

**What's in the exported file** ([project-files/](project-files/)):

| Part | Generated by | Contents |
|---|---|---|
| `info`, `host`, `basePath`, `paths` | API Gateway, automatically | Title and version, the `/prod` base path, `/users` with GET, the Lambda integration (`aws_proxy`) |
| `x-amazon-apigateway-documentation` | **You** | The custom `description` and `baseURL` |

The generated parts are accurate but generic. The written parts can answer specific questions and guide users in a friendlier way.

## Build order and cleanup

1. Create a **Lambda function** `RetrieveUserData`: runtime **Node.js**, architecture **x86_64**.
2. Paste the code into the function's page and **Deploy** it.
3. In API Gateway, create a **REST API** (`UserRequestAPI`) with a **Regional** endpoint.
4. Create a **resource** with path `/users`.
5. Create a **GET** method: integration type **Lambda function**, turn on **Lambda proxy integration**, choose `RetrieveUserData`.
6. **Deploy API** to a **new stage** called `prod`, then copy the **invoke URL**.
7. Visit `<invoke URL>/users` and get an error (no DynamoDB table yet) ⚠️
8. **Documentation:** create a documentation part (type **API**) with a JSON `description` and `baseURL`, then **publish** it to `prod`.
9. **Export** the stage as **Swagger** (JSON) to see the generated and written documentation.

**Cleanup:** delete the API in API Gateway, delete the Lambda function, then delete its CloudWatch log group (`/aws/lambda/RetrieveUserData`).

> **Check first:** the three-tier series continues. If the next project builds on this API and function, keep them until it's done.

---

## Self-quiz

<details><summary>1. What is AWS Lambda, and why is it cost-efficient?</summary>

A serverless service that runs your code without you managing servers. It runs only when needed, so you don't pay for idle time, and it scales automatically.
</details>

<details><summary>2. What is a runtime, and which one did you use?</summary>

The environment your code runs in, with the dependencies it needs. This project used **Node.js**.
</details>

<details><summary>3. What does Node.js make possible?</summary>

Running JavaScript on the **server side**, not just in the browser, so the same language can be used for both front end and backend.
</details>

<details><summary>4. Lambda is serverless, so why do you pick an architecture?</summary>

Serverless means you don't manage the servers, but the code still runs on physical hardware. The architecture (e.g. x86_64) is the processor type it runs on, and AWS manages it for you.
</details>

<details><summary>5. What is an API?</summary>

A way for software systems to talk to each other, like a messenger carrying requests and responses.
</details>

<details><summary>6. What role does API Gateway play for Lambda?</summary>

It's the **front door**. It receives requests, forwards them to Lambda and sends Lambda's response back to the user.
</details>

<details><summary>7. Why not let users call the Lambda function directly?</summary>

Lambda on its own lacks API management features. API Gateway adds authentication and authorization, plus API management such as request routing.
</details>

<details><summary>8. Name the three API types and what each is for.</summary>

**REST** (standard web API, full features), **HTTP** (simpler, cheaper routing) and **WebSocket** (real-time, two-way communication).
</details>

<details><summary>9. Name the three endpoint types.</summary>

**Regional** (one Region, used here), **Edge-Optimized** (global, through CloudFront edge locations) and **Private** (only inside your VPC).
</details>

<details><summary>10. What's the difference between a resource and a method?</summary>

A **resource** is an endpoint/path, like `/users`. A **method** is the action you perform on it, like GET.
</details>

<details><summary>11. What do GET, POST, PUT and DELETE do?</summary>

Retrieve, add, update and remove data.
</details>

<details><summary>12. What is an integration type?</summary>

The backend service that fulfils the API request. It decides how API Gateway passes the request to the backend and handles the response. Here it was a Lambda function.
</details>

<details><summary>13. What does Lambda proxy integration change?</summary>

API Gateway passes the **entire** request (headers, query parameters, path parameters, body) straight to Lambda instead of reformatting it. The function has to parse the request and return a properly shaped response itself.
</details>

<details><summary>14. With proxy integration, what must the Lambda function return?</summary>

An object with `statusCode`, optional `headers`, and `body` as a **string**. Otherwise API Gateway returns `502 Bad Gateway`.
</details>

<details><summary>15. What is a stage, and why have more than one?</summary>

A named, deployed version of the API, such as `dev`, `test` or `prod`. Separate stages let you build and test changes without affecting live users.
</details>

<details><summary>16. You changed the API in the console, but the invoke URL still behaves the old way. Why?</summary>

You haven't **redeployed** to the stage. Console changes only go live after a new deployment.
</details>

<details><summary>17. Why did the invoke URL return an error at first?</summary>

The DynamoDB table the function reads from hadn't been created yet. The data tier was still missing.
</details>

<details><summary>18. What's the difference between Swagger and OpenAPI?</summary>

Both describe REST APIs. Swagger 2.0 is the older format. After Swagger was donated to the OpenAPI Initiative, the next version was released as **OpenAPI 3**.
</details>

<details><summary>19. Why publish documentation to a stage?</summary>

So the documentation matches the version of the API running on that stage (here, `prod`).
</details>

<details><summary>20. Which part of a three-tier architecture did this project build?</summary>

The **logic tier**: the backend code (Lambda) plus the front door that connects users to it (API Gateway).
</details>
