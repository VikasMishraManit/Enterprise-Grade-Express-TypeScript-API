## ⬢ New Section : Setup basic project 

1) set up node js project
```
npm init -y
```

2) install express
```
npm install express
```

3) Now , add more dependencies to help us in the project . Install these as dev dependencies  
```
npm install -D ts-node
```
This dependency ts-node automatically compiles the ts for us.

4) when we installed express , a lot of js code was also installed so we need to define types for those
```
npm install -D @types/express
npm install -D @types/node
```

5) install nodemon library
```
npm install -D nodemon
```

See all these installations in the package.json file



## ⬢ New Section : Configure the typescript 

We will define how typescript should behave in this project 

1) First install typescript as dev dependency
```
npm install -D typescript
```

2) Make the tsconfig.json  file  (manually or by code )
```
npx tsc --init
```



## ⬢ New Section : Configuring the tsconfig file

1) include : directories to include (for ts compliation)
2) exclude : directories to exclude (for ts compilation)

3) file config
```ts
{
  "compilerOptions": {
  
    "target": "es2016",                                  /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
    "module": "commonjs",                                /* Specify what module code is generated. */
    "rootDir": "./src",                                     /* Specify the root folder within your source files. */                            
    "outDir": "./dist",                                   /* Specify an output folder for all emitted files. */            /* Allow 'import x from y' when a module doesn't have a default export. */
    "esModuleInterop": true,                             /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */
    "forceConsistentCasingInFileNames": true,            /* Ensure that casing is correct in imports. */

    /* Type Checking */
    "strict": true,                                      /* Enable all strict type-checking options. */                            /* Ensure 'use strict' is always emitted. */
    "noUnusedLocals": true,                              /* Enable error reporting when local variables aren't read. */                      /* Skip type checking .d.ts files that are included with TypeScript. */
    "skipLibCheck": true                                 /* Skip type checking all .d.ts files. */
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ]
}
```



## ⬢ New Section : Running the Typescript code 

1) Define a basic server in src->server.ts file
```ts
import express from 'express';

const app = express();

const PORT = 3000

app.get('/ping' , (req , res)=>{
  res.send('pong')
})

app.listen(PORT, ()=>{
    console.log(`Server is running on http://localhost:${PORT}`);
    console.log(`Press CTRL+C to stop the server`);
})
```

2) Directly execute the file ( server will start running)
```
npx ts-node src/server.ts 
```

3) Create a dist folder , inside that folder we will be having alll the ts->js compiled files. 
```
npx tsc
```
Now  server.js file will be there (with type checking) 
To run this file 
```
node dist/server.js 
```
This js code will not be having a lot of baggage and we can directly ship this js code 



## ⬢ New Section : Scripts for running the code 

1) In the package.json file
```
 "scripts": {
    "start": "npx ts-node src/server.ts",
    "dev": "nodemon src/server.ts"
  },
```

2) To run this command
```
// for start or test 
npm start

// for other commands
npm run dev
```


## ⬢ New Section : Git setup

1) Make a .gitignore file . put following thins there
```
node_modules/
dist/
```
2) Setup the repo 
```
git init
git add .
git commit -m "add commit"
git remote add origin ""
git checkout -b master
git push origin master
```



## ⬢ New Section : Need for  .env file

1) Handling sensitive and configurable info (say port changes then application will have to redeployed). To handle this we will make a env variables. 
Env variables are stored in os level and any process can access these ex: type env in terminal to see all these.

2) If we change these varaibales , now we just have to restart the server and the server can pick them again . So , we dont have to redeploy the server. 

3) To provide extra layer of security in these env variables , people use key vaults like aws key vault , azure key vault etc. 


## ⬢ New Section : Creating  .env file

1) To manage these env variables there is a nodejs package
```
npm i dotenv
```
2) Now create a .env file inside the project only and add it to .gitignore file
```
PORT=3001
```

## ⬢ New Section : Need of config layer 

A "config layer" is a design pattern used to centralize application settings, environment variables, and external service credentials. Instead of scattering process.env calls throughout your codebase, you create a dedicated module to manage these values. 

Key Purposes of a Config Layer - 
1) Centralization: All settings (database URLs, API keys, port numbers) are managed in one location.
2) Validation: Ensures required environment variables exist and are in the correct format (e.g., converting a port string to a number) before the app starts.
3) Ease of Testing: Simplifies mocking configuration values during unit and integration tests.

## ⬢ New Section : Creating  config layer 

1) Create a folder src-> config -> index.ts . This file will have all the basic configuration logics. 
```ts
import dotenv from 'dotenv';

function loadEnv (){
dotenv.config();
}

export default loadEnv;
```

2) dotenv.config(): it will load all the env variables from the .env file to the config file when server is running . When server stops then it doesn't get loaded. 

3) Now , in the server.ts file import and load these and use process.env.PORT instead of PORT. 
```ts
import express from 'express';
import loadEnv from './config/index.js';

const app = express();



app.get('/ping' , (req , res)=>{
  res.send('pong')
})

loadEnv();
console.log(`Environment variables loaded`);

app.listen(process.env.PORT, ()=>{
    console.log(`Server is running on http://localhost:${process.env.PORT}`);
    console.log(`Press CTRL+C to stop the server`);
    
})
```



## ⬢ New Section : Cleaning the code 

1) In index.ts we will create a object that has access to all the env variables , so instead of writing process.env everywhere we can use that object . This will allow us to have centralized control over the config variables. 

2) Define the type for that object 
```
type ServerConfig = {
    PORT : number
}
```
3) Now make a object using this type .
```ts
import dotenv from 'dotenv';

type ServerConfig = {
    PORT : number
}

export function loadEnv (){
dotenv.config();
}

export const serverConfig : ServerConfig = {
    // process.env.PORT is a string , so convert it to number 
    PORT : Number(process.env.PORT) || 3001
}

```

4) Now the server.ts file will look like this (Note the import has been changed and process.env has been replaced)
```ts
import express from 'express';
import {loadEnv , serverConfig} from './config';

const app = express();



app.get('/ping' , (req , res)=>{
  res.send('pong')
})

loadEnv();
console.log(`Environment variables loaded`);

app.listen(serverConfig.PORT, ()=>{
    console.log(`Server is running on http://localhost:${serverConfig.PORT}`);
    console.log(`Press CTRL+C to stop the server`);
    
})
```

5) Checking : Remove the port from the .env file and re-run the server
```
npx ts-node src/server.ts 
```

6) If we put the port value say PORT=3000 , we will still see the fallback port value. This is because 
there is a issue .




## ⬢ New Section : Fixing the port issue 

1) We are importing these things (import {loadEnv , serverConfig} from './config';) but at this point of time the
env variables have not been loaded yet . They will load only when we will call (loadEnv();)

2) Remove this (loadEnv();) from the server.ts file and call this in the index.ts file only
Change that we will have to make in the index.ts file
```
 function loadEnv (){
dotenv.config();
}

loadEnv();
```

and change in the server.ts file
```
import { serverConfig} from './config';

// and remove the loadEnv() line
```

## ⬢ New Section : Need for the controller layer

1) Controller after the validators are the first function that hadles our request.
In server.ts file
```ts
app.get('/ping', (req, res) => {
  res.send('pong');
});
Receives an HTTP request
Sends an HTTP response
That is exactly what a controller does.
```

2) Make a file src/controllers/ping.controller.ts
```ts
import { Request, Response } from "express"

export const pingHandler = (req:Request , res:Response)=>{
  res.send('pong')
}
```

3) In the server.ts file
```ts
/* this will get replaced
app.get('/ping', (req, res) => {
  res.send('pong');
});
*/

app.get('/ping' , pingHandler)

// also import the pingHandler
// import { pingHandler } from './controllers/ping.controller';
;
```

4) If we clearly see then , this line (app.get('/ping' , pingHandler)) is acting like a router

## ⬢ New Section : Need for the routing layer

1) Routing Layer : It maps HTTP request with the correct controllers 

It decides:
- Which URL is called (/users, /orders/:id). 
- Which HTTP method (GET, POST, PUT, DELETE). 
- Which controller should handle it. 

2) For this : app.get('/ping' , pingHandler);
URL : /ping
HTTP : GET
Controller : ping controller


## ⬢ New Section : Brute force way of making the router 

1) Create a file src/routers/ping.router.ts
```ts
import { Express } from "express";
import { pingHandler } from "../controllers/ping.controller";

export function createPingRouter (app : Express){
     app.get('/ping' , pingHandler);
}
```

2) In the server.ts file 
```ts
// app.get('/ping' , pingHandler);
createPingRouter(app)
```

3) To check this : Start the server and in new terminal send this 
```
curl -X GET http://localhost:3000/ping
```

4) Issue with this approach is that we are passing app to a seperate function that can even try to modify the app object.  
So , the recommended mechanism is the express router mechanism


## ⬢ New Section : Express router mechanism 

1) In above approach we are defining the routes on the app object itself . But now using express router mechanism we will define the routes on a router object and then we will attach that router object to the app object.

2) In the ping.router.ts file
```ts
import  express  from "express";
import { pingHandler } from "../controllers/ping.controller";

const pingRouter = express.Router();

pingRouter.get('/ping' , pingHandler);

export default pingRouter;
```

3) And in the server.ts file (use app.use(pingRouter))

```ts
import express from 'express';
import { serverConfig} from './config';
import pingRouter from './routers/ping.router';

const app = express();



// app.get('/ping' , pingHandler);
// createPingRouter(app)


app.use(pingRouter);

console.log(`Environment variables loaded`);

app.listen(serverConfig.PORT, ()=>{
    console.log(`Server is running on http://localhost:${serverConfig.PORT}`);
    console.log(`Press CTRL+C to stop the server`);
    
})
```

4) Here we are registering all the routes that are being handled by the pingRouter to the app object .
You can check this by sending the curl request


## ⬢ New Section : Middlewares

1) Middleware handles cross-cutting concerns that should NOT live in controllers or routes.

2) Definition: A middleware is a function that has access to the request object (req), the response object (res), and the next middleware function in the application’s request-response cycle.

3) Examples of middlewares:
- Logging Middleware: Logs details about each incoming request.
- Authentication Middleware: Verifies user authentication before allowing access to certain routes. 
- Error Handling Middleware: Catches and processes errors that occur during request handling.

4) Routing - > Middlwware 1 -> Middleware 2 -> Controller -> Response
ex; app.get(/ping , startMiddleware)

5) This is how we chain the middlewares :
 pingRouter.get('/ping' , middleware1 , middleware2 , pingHandler);



## ⬢ New Section : Use Case of Middlewares

1) Sepration of concerns
ex : for these kind of validation chaining we make use of middlewares

request -> validateRequestBody -> validateAuthentication -> validateAuthorization -> operationController


2) In server.ts file
```ts
// any middleware registered with app.use is going to be used in every request
app.use(pingRouter);
```

3) Bcz of the middlewares , express router gives us the flexibilty to seggregate the urls in the request. 
```ts
// in server.ts file , if any request has url /ping (its type doesnt matter get/post etc)
// pass it to the ping Router
app.use('/ping', pingRouter);

// now in ping router.ts file
pingRouter.get('/' , pingHandler); // if after that url is empty and it is a get request then pass it to pingHandler. 

// another example if remaining url has /health and is a get request then handle it as given
pingRouter.get('/health' , (req,res)=>{
  res.status(200).send('OK');
})
```



## ⬢ New Section : API Versioning

1) If the request URL starts as 
```ts
// if request url is as given below handle it with v1Router 
app.use('/api/v1' , v1Router);
``` 

2) Folder : Router -> v1 -> index.router.ts
```ts
import express from 'express'
import pingRouter from './ping.router';

const v1Router = express.Router();

// write the logic of v1Router here

v1Router.use('/ping' , pingRouter);

// move the ping.router.ts file to v1 folder and update the imports

export default v1Router ; 

```

3) Now we can make one more folder v2 similar to v1

4) See only at the last middleware (pinghandler) we are using app.get or app.post . Rest we are just using app.use 



## ⬢ New Section : Accessing query Params and Request Body


1) In the ping handler function
```ts
import { Request, Response } from "express"

export const pingHandler = (req:Request , res:Response)=>{

  console.log('request body is : ' , req.body);
  console.log('request query is : ' , req.query);
  res.send('pong')
}
```

2) Now send the query param and req body ( raw json) through the postman.
Request body can be sent in both get and post request
```

// query param
// http://localhost:3000/api/v1/ping?age =23&city=bengaluru

// request body 
{
    "name" : "Vikas"
}

// we are seeing the query params but the request body is still undefined
request body is :  undefined
request query is :  [Object: null prototype] { 'age ': '23', city: 'bengaluru' }

```

3) Reason : Both the query params and URL params are written as string . So they are deterministics (their types) , so express also knows their data types . So it can parse it. 

However , the request body can be of any types (json in rest , xml in soap etc) , so we have to tell express about the different type of data that we are going to read. 

This concept is called as serialization and de-serialization. 

4) To fix this , in the server.ts file 
```ts
// whatever request body is coming , I am trying to convert it to json
app.use(express.json());

// now the output is
request body is :  { name: 'Vikas' }
request query is :  [Object: null prototype] { 'age ': '23', city: 'bengaluru' }

// if text is coming
app
```

5) For the url encoded data. 

The url cannot have anything inside it (like comma). But using allowed characters (using these characters we will make a commma ) we can use this (	its url encoding is this : %2C)

We can also send this type of url encoded data in the express .

```ts
// for this the middleware is this
app.use(express.urlencoded({ extended: true }));

// extended = true -> means qs library for pasrsing
// extended = false -> we will use query string library
```

6) Query params start after ? . However , for Rreading URL Params When you want to send the data in the url params , you have to tell the express js that this part of the url is varaible

pingRouter.get('/:id/comments' , pingHandler)(write it where final request have been mentioned). 
That colon part is the variable part , comments is a constant part (since no colon)


## ⬢ New Section : Need for ZOD Validation

1) JSON's are not type safe , so we cannot enforce a contract. So we use the validation layer. 

2) Request body can be very heavy , so we will be requiring different kinds of check for that. So we will need a smarter soluton rather than manually looking at it. 

3) For doing this we will be using ZOD. 



## ⬢ New Section : Integrating ZOD in the project

1) Install zod. ZOD has a concept of schema that means for every request that is coming we need to define the schema for that request. 
```
npm i zod
```

2) For different types of request body , we are going to maintain the schema. Example schema for create user request , schema for create post request etc. 

To create the schema validators ->ping.validator.ts (create the schema in this file)

```ts
import {z} from "zod";


export const pingSchema = z.object({
   message : z.string().min(1)
})
```


3) Setup : src ->validators ->index.ts. Here we will have a zod schema to validate the request body. And we are going to return the middleware. 

-> If we will call this function and pass ZodSchema then it is going to return a middleware
```ts
import { NextFunction, Request, Response, response } from "express";
import { ZodObject, ZodRawShape } from "zod";


// -> we are going to use this function as a middleware in our routes 
// to validate the request body against a given Zod schema

export const validateRequestBody = (schema: ZodObject<ZodRawShape>) => {

    // return async function to be used as middleware

    return async (req : Request , res: Response , next : NextFunction) => {
        try {

            // validate the request body against the schema
            await schema.parseAsync(req.body);
            
            // if validation is successful, call the next middleware or route handler
            next();

        } catch (error) {  

             res.status(400).json({
                message: "Invalid request body",
                success: false,
                error: error
            });
            
        }
    }
}

export const validateQueryParams =  (schema: ZodObject<ZodRawShape>) => {
    return async (req: Request, res: Response, next: NextFunction) => {
        try {

            await schema.parseAsync(req.query);
            console.log("Query params are valid");
            next();

        } catch (error) {
            // If the validation fails, 

            res.status(400).json({
                message: "Invalid query params",
                success: false,
                error: error
            });
            
        }
    }
}
```


4) Now go in the ping.router.ts file and I want to validate  requests with the same incoming request body. 
 ```ts
 // req 1 : 
pingRouter.get('/' , pingHandler);

// to validate this we do this
pingRouter.get('/' , validateRequestBody(pingSchema) ,pingHandler);

// let us say we want to validate with userSchema next time
pingRouter.get('/' , validateRequestBody(userSchema) ,pingHandler);

// this has made code quality much better. We have a function validateRequestBody which takes a schema
// pasres it and return result accordingly 


```





## ⬢ New Section : Adding Error Handling. 

1) Errors that occur in synchronous code inside route handlers and middleware require no extra work. If synchronous code throws an error, then Express will catch and process it
```ts
app.get('/', (req, res) => {
  throw new Error('BROKEN') // Express will catch this on its own.
})

```

2) For errors returned from asynchronous functions invoked by route handlers and middleware, you must pass them to the next() function, where Express will catch and process them. Starting with Express 5, route handlers and middleware that return a Promise will call next(value) automatically when they reject or throw an error


3) Example 
```ts
app.get('/user/:id', async (req, res, next) => {
  const user = await getUserById(req.params.id)
  res.send(user)
})
```
If getUserById throws an error or rejects, next will be called with either the thrown error or the rejected value. If no rejected value is provided, next will be called with a default Error object provided by the Express router.

If you pass anything to the next() function (except the string 'route'), Express regards the current request as being an error and will skip any remaining non-error handling routing and middleware functions.


## ⬢ New Section : Handling  synchronous error

1) In the ping.controller.ts file throw a error
```ts
import { Request, Response } from "express"

export const pingHandler = (req:Request , res:Response)=>{

  throw new Error("This is a test error for testing the error handling middleware");
  
  // res.status(200).json({
  //   message : "pong",
  //   success : true
  // })
}
```

2) Now 
```
npm run dev 

// in the postman make a get request to 
http://localhost:3000/api/v1/ping

// also since we are validating , and validation expects a message 
// in postman -> body -> raw
// write this message
//{"message":"hello"}

// now make the request

// ans we will see the error
// 500 Internal Server Error
```

## ⬢ New Section : Handling  Asynchronous error

1) Express says if we get an async error , we just have to pass it to the next middleware . This next middleware is being built by express on its own and it handles the logic for it. 

```ts
import { NextFunction, Request, Response } from "express"
import fs from 'fs';

export const pingHandler = (req:Request , res:Response , next : NextFunction)=>{

  fs.readFile('sample.txt' , 'utf-8' , (err,data)=>{
    if(err){
      console.error("Error reading file:", err);
      next(err); // defualt error handling middleware will handle this error
    }

    res.status(200).json({
      message : "Pong",
      success : true,
      data : data
    })
  })  
}
```
## ⬢ New Section : Custom error handling

1) Let us say we want to send a custom error response for a error of a controller . And also for a lot of controller we want to send this same custom error response. 
So we are going to make a central error handler 

2) Add this generic error handler after all the routers in server.ts file
```ts

app.use('/api/v1' , v1Router);
app.use('/api/v2' , v2Router);

// default error handling middleware
app.use(generalErrorHandler);
```

3) Code for generic error handler : src/middlewares/error.middleware.ts
```ts
import { NextFunction, Request, Response } from "express";

// argument of this should be the error object we want to handle
// error handling function has 4 arguments instead of 3 like normal middleware functions
export const generalErrorHandler = (err: any, req: Request, res: Response, next:NextFunction) => {

    res.status(501).json({
        message: "Something went wrong",
        success: false
    });
}
```

3) And in the pingcontroller.ts , call the next error . Now this default next error handler of express will be overwritten by our custom error handler
```ts
import { NextFunction, Request, Response } from "express"
import fs from "fs/promises";

export const pingHandler = async (req:Request , res:Response , next : NextFunction)=>{

   try {
      await fs.readFile("sample");
      res.status(200).json({
        message: "Pong",
        success: true
      });
   } catch (error) {
          next(error);
   }
}
```

4) As expected we see this response in postman with the status code of 501 as defined by us in the custom error handler
```
{
    "message": "Something went wrong",
    "success": false
}
```

5) For express version 5 or more , if that async function fails it automaticaaly calls the error handler
```ts
// insted of this
  try {
      await fs.readFile("sample");
      res.status(200).json({
        message: "Pong",
        success: true
      });
   } catch (error) {
          next(error);
   }
}

// we can directly write this

await fs.readFile("sample"); -> if this fails automatically error handler is called 

      res.status(200).json({
        message: "Pong",
        success: true
      });
```


## ⬢ New Section : Improving the error response

1) File : utils/errors/app.error.ts file 
```ts
// src/utils/errors/app.error.ts

/**
 * AppError
 *
 * A custom error class used throughout the application.
 *
 * Why do we need this?
 *
 * The default JavaScript Error object only contains:
 *  - message
 *  - name
 *  - stack
 *
 * But in an API, we also need an HTTP status code
 * so that our error middleware knows what response
 * to send to the client.
 *
 * Example:
 *
 * throw new AppError("User not found", 404);
 *
 * Instead of:
 *
 * throw new Error("User not found");
 */
export class AppError extends Error {

  /**
   * HTTP status code to be returned
   *
   * Examples:
   * 400 -> Bad Request
   * 401 -> Unauthorized
   * 403 -> Forbidden
   * 404 -> Not Found
   * 500 -> Internal Server Error
   */
  public readonly statusCode: number;

  /**
   * Indicates whether this is an expected
   * application error or an unexpected system error.
   *
   * Examples:
   * User not found -> true
   * Invalid token -> true
   * Database crash -> false
   */
  public readonly isOperational: boolean;

  constructor(
    message: string,
    statusCode: number = 500,
    isOperational: boolean = true
  ) {

    /**
     * Calls the parent Error constructor.
     *
     * Without this:
     * err.message would not be set correctly.
     */
    super(message);

    /**
     * Save additional information
     * on the error object.
     */
    this.statusCode = statusCode;
    this.isOperational = isOperational;

    /**
     * Fixes prototype chain.
     *
     * Required when extending built-in classes
     * like Error in TypeScript.
     *
     * Without this:
     *
     * err instanceof AppError
     *
     * may return false.
     */
    Object.setPrototypeOf(this, new.target.prototype);

    /**
     * Captures the exact stack trace
     * from where the error was thrown.
     *
     * Example:
     *
     * Error: User not found
     *   at UserService.getUser()
     *   at UserController.getUser()
     */
    Error.captureStackTrace(this);
  }
}
```

2) In typescript there is one more way for this : make interface , use that in error middleware and then in
ping controller (create app error object using the interface)
```ts
1) Interfaces can be used for object oriented. 
2) They can act as a contracts ( We can make a class implement this contract). 
3) Interface vs class . Interface allows multiple inheritance (bcz of polymorphism)

//1: app.error.ts
export interface AppError extends Error {
    statusCode: number;
}

//2: error.middleware.ts
import { NextFunction, Request, Response } from "express";
import { AppError } from "../utils/errors/app.error";

// argument of this should be the error object we want to handle
// error handling function has 4 arguments instead of 3 like normal middleware functions
export const generalErrorHandler = (err: AppError, req: Request, res: Response, next:NextFunction) => {

    res.status(err.statusCode).json({
        message: err.message,
        success: false
    });
}

//3 : ping.controller.ts
 import { NextFunction, Request, Response } from "express"
// we will use promise version because it is easier to work with async/await and 
// it is more modern than the callback version of fs module
import fs from "fs/promises"; 
import { AppError } from "../utils/errors/app.error";

export const pingHandler = async (req:Request , res:Response , next : NextFunction)=>{

  // // for express version 5 or more if our async function throws an error 
  // // it will be automatically caught and passed to the error handling middleware
  // await fs.readFile("sample"); // if it fails it will throw an error and it will be caught by the error handling middleware

  //     res.status(200).json({
  //       message: "Pong",
  //       success: true
  //     });

  try {
     await fs.readFile("sample"); 
     res.status(200).json({
        message: "Pong",
        success: true
      });
  } catch (error) {
    // make apperror object
    const appError: AppError = {
        statusCode: 500,
        message : "Internal Server Error",
        name: "InternalServerError",
    }
    throw appError; // this error will be caught by the error handling middleware
    
  }
}
```

3) Test in the postman
```
http://localhost:3001/api/v1/ping
body->raw-> {"message": "hello"}

response 
{
    "message": "Internal Server Error",
    "success": false
}

and we will see 500 internal server error (what we coded)
```

4) We can make it even more cleaner
```ts
//1: in app.error.ts file
export interface AppError extends Error {
    statusCode: number;
}

export class InternalServerError  implements AppError {
    statusCode: number;
    message: string;
    name: string;

    constructor(message: string) {
        this.statusCode = 500;
        this.message = message;
        this.name = "InternalServerError";
    }
}

// we can create more custom error classes like NotFoundError, BadRequestError etc. 
// by implementing the AppError interface and setting the appropriate status code and message.




//2:  in ping.controller.ts file
import { NextFunction, Request, Response } from "express"
// we will use promise version because it is easier to work with async/await and 
// it is more modern than the callback version of fs module
import fs from "fs/promises"; 
import { InternalServerError } from "../utils/errors/app.error";

export const pingHandler = async (req:Request , res:Response , next : NextFunction)=>{


  try {
     await fs.readFile("sample"); 
     res.status(200).json({
        message: "Pong",
        success: true
      });
  } catch (error) {
    // we will throw internal server error which then will be caught by the error handling middleware 
    // and sent to the client as a response
    throw new InternalServerError("Something went wrong !!!"); // this will be caught by the error handling middleware
  }
}

```






## ⬢ New Section : Adding production grade loggers

1) Use case : Let us say somebody made a payment on airbnb , the payment got deducted but they received errors . As a engineer , to look into the problem , we have to know few things. 

- when the request was made
- what procedures were done right
- what went wrong with it etc. 

For all these logging is very necessary things (on call engineers depend on this )

2) Why console.log() will not work ? : Because that will be visible for only that session

3) There are many logging libraries for us to use like morgan , pino , winston etc. 
Logging Library we are going to use : Winston

## ⬢ New Section : Integrating Winston logging 

1) Install winston and prepare a logging object using winston . This object will help us in logging
```
npm i winston
```

2) We will configure winston
```ts
import winston from "winston";

// create the logger object with the desired configuration
const logger = winston.createLogger({
// let us now configure this object 

// 1: format : what should logs show ex: timestamp, log level, message etc.
format: winston.format.combine(
    winston.format.timestamp({format: 'MM-DD-YYYY HH:mm:ss'}), // this will add a timestamp to each log entry in the specified format
    winston.format.json(), // this will log in JSON format, which is useful for structured logging and easier parsing. 
  // define the custom print
    winston.format.printf(({ timestamp, level, message , ...data }) => {
       const output = {level , message , timestamp, data};
       return JSON.stringify(output); // this will convert the log entry to a JSON string
    })
),

// 2: transports : where should the logs be stored or displayed
transports: [
    new winston.transports.Console(), // this will log to the console
]
});

// export the logger to be used in other parts of the application
// we do default export here because we want to import it with any name in other files without using curly braces
export default logger;
```

3) Testing the logging 
```ts
// go to server.ts 
import logger from './config/logger.config';

// code ....


app.listen(serverConfig.PORT, ()=>{
    console.log(`Server is running on http://localhost:${serverConfig.PORT}`);
    logger.info(`Press CTRL+C to stop the server`);
    
})
```

```
// run : npm run dev and see this output 

Environment variables loaded
Server is running on http://localhost:3000
{"level":"info","message":"Press CTRL+C to stop the server","timestamp":"06-03-2026 15:28:02","data":{}}
```


## ⬢ New Section : Concept of Correlation id 

1) Problem Statement : User 1 made a request and simultaneously user 2 also made a request to a end point. How are we going to ensure that logs of user 1 and user 2 are getting differentiated. 

2) To do this we use correlation id 
```
// suppose logs look like this 
{"correlationId":"abc123","message":"Request Started"}
{"correlationId":"xyz789","message":"Request Started"}
{"correlationId":"abc123","message":"User Found"}
{"correlationId":"xyz789","message":"Password Invalid"}

// say we want to search all log of abc123 
grep abc123 app.log

// output 
{"correlationId":"abc123","message":"Request Started"}
{"correlationId":"abc123","message":"User Found"}

// at big scale we have this flow
Request
   |
Correlation ID
   |
Logs
   |
Central Log Store
   |
Search Dashboard
```

## ⬢ New Section : Implementing Correlation id 

1) Prepare a middleware for attaching the UUID as correlation id. 
```ts
File name : src/middlewares/correlation.middleware.ts

import { NextFunction, Request, Response } from 'express';
import {v4 as uuidv4} from 'uuid';

// function to generate a unique correlation ID for each request
export const attachCorrelationId = (req:Request, res:Response, next:NextFunction) => {
    // generate a unique correlation ID using uuid library
    const correlationId = uuidv4();

   // in typescript, we need to tell the compiler that we are adding a new property to the request object, 
   // so we will use type assertion to do that
   // attach the correlation ID to the request headers so that it can be accessed in the route handlers and other middlewares
    req.headers["x-correlation-id"] = correlationId;

    // we can also attach the correlation ID to the response headers so that it can be accessed by the client
    // res.setHeader("x-correlation-id", correlationId);

    // call the next middleware or route handler
    next();
}



// in the server.ts file before any request attach it
import { attachCorrelationId } from './middlewares/correlation.middleware';
app.use(attachCorrelationId)

```

2) Let us test it 
```ts
// File : validators/index.ts file
export const validateRequestBody = (schema: ZodObject<ZodRawShape>) => {

    // return async function to be used as middleware

    return async (req : Request , res: Response , next : NextFunction) => {
        try {
            // adding log
            logger.info("Validating request body against schema" , {correlationId : req.headers["x-correlation-id"]});

            // validate the request body against the schema
            await schema.parseAsync(req.body);

            // one more log 
            logger.info("Request body is valid" , {correlationId : req.headers["x-correlation-id"]});
            
            // if validation is successful, call the next middleware or route handler
            next();

        } catch (error) {  

             res.status(400).json({
                message: "Invalid request body",
                success: false,
                error: error
            });

        }
    }
}


// and in ping.controller.ts
import { NextFunction, Request, Response } from "express"
// we will use promise version because it is easier to work with async/await and 
// it is more modern than the callback version of fs module
import logger from "../config/logger.config";

export const pingHandler = async (req:Request , res:Response , next : NextFunction)=>{

  logger.info("Received ping request" , {correlationId : req.headers["x-correlation-id"]});
  res.status(200).json({
    message: "pong",
    success: true
  });

}
```

and now 
```
// run npm run dev and send request to http://localhost:3000/api/v1/ping
// output in terminal is 
Server is running on http://localhost:3000
{"level":"info","message":"Press CTRL+C to stop the server","timestamp":"06-03-2026 18:41:00","data":{}}
{"level":"info","message":"Validating request body against schema","timestamp":"06-03-2026 18:41:06","data":{"correlationId":"cbc4ca29-31c3-4628-819c-dffe6760e6d2"}}
{"level":"info","message":"Request body is valid","timestamp":"06-03-2026 18:41:06","data":{"correlationId":"cbc4ca29-31c3-4628-819c-dffe6760e6d2"}}
{"level":"info","message":"Received ping request","timestamp":"06-03-2026 18:41:06","data":{"correlationId":"cbc4ca29-31c3-4628-819c-dffe6760e6d2"}}

// we can see that for this request correlation id is same

```



## ⬢ New Section : Imporving this Correlation id middleware with AsyncLocalStorage

1) This correlation id works well uptil we are using the network request (process is calling rest api) i.e. a path has request flow through routers , controllers etc. What if we had a process that directly calls utility function/service layer etc , then this correlation id will not be attached to it. 

2) 
```
When a request enters your Express app, you often create data that should be available throughout the entire request lifecycle, such as a correlationId, userId, or tenantId. The traditional way is to pass these values through every function call:

Controller(correlationId)
  -> Service(correlationId)
      -> Repository(correlationId)

This becomes messy because many functions receive parameters they don't actually need. AsyncLocalStorage solves this by creating a request-specific storage area. In a middleware, you create a context:

asyncLocalStorage.run(
  { correlationId: "abc-123" },
  () => next()
);


From that point onward, any code executed as part of that request can access the same data using:

const store = asyncLocalStorage.getStore();
console.log(store?.correlationId);

even deep inside services, repositories, helpers, or after multiple await calls. Node internally tracks which async operations belong to which request and automatically restores the correct context when execution resumes. 

Think of it as giving every request its own invisible backpack containing metadata (correlationId, userId, etc.). Any function handling that request can open the backpack and read the data without needing the original req object or manually passing parameters through every layer. This is why modern Node.js applications use AsyncLocalStorage for logging, tracing, observability, and request-scoped data.

```

3) Let us implement it . 
```ts
// Step1 : Function to retrieve correlation id 

// File name : src/utils/helpers/request.helpers.ts
import {AsyncLocalStorage} from "async_hooks";

// define type
type AsyncLocalStorageType = {
    correlationId: string;
}

// create an instance of AsyncLocalStorage to store the correlation ID for each request
export const asyncLocalStorage = new AsyncLocalStorage<AsyncLocalStorageType>();

// function to get correlation ID from the AsyncLocalStorage
export const getCorrelationId = ()=> {
    const asyncstore = asyncLocalStorage.getStore();
    return asyncstore ? asyncstore.correlationId : 'unknown-error-while-getting-correlation-id';
}


// Step2 : Ensure we are adding this correlation id 
// File name : src/middlewares/correlation.middleware.ts
import { NextFunction, Request, Response } from 'express';
import {v4 as uuidv4} from 'uuid';
import { asyncLocalStorage } from '../utils/helpers/request.helpers';

// function to generate a unique correlation ID for each request
export const attachCorrelationId = (req:Request, res:Response, next:NextFunction) => {
    // generate a unique correlation ID using uuid library
    const correlationId = uuidv4();

    req.headers["x-correlation-id"] = correlationId;

   /*
    We will use the asyncLocalStorage to store the correlation ID for each request,
    this will allow us to access the correlation ID in any part of the code that is executed as part of the request,
    even if it is executed in a different async context, such as in a setTimeout or in a database query callback,we will 
    still be able to access the correlation ID using the getCorrelationId function that we defined in the request.helpers.ts file.

   */
    asyncLocalStorage.run({ correlationId: correlationId }, () => {
        // call the next middleware or route handler inside the asyncLocalStorage.run callback to ensure that the correlation ID is available in the async context
        next();
    });

};
```



3)  We will not maually attach correlation id in logger(in each log line). Whenever someone will create log (log.info , log.error) we will call getcorrelationId() function. 
In this function we will fetch storage and inside this storage if we have a correlation id we will print that. 

```ts

// 1 : in config/logger.config.ts
// change the output from
//        const output = {level , message , timestamp, data};
// to 
const output = {level , message , timestamp,correlationId : getCorrelationId(), data};


// first in controller/ping.controller.ts
/*
  logger.info("Received ping request" , {correlationId : req.headers["x-correlation-id"]});
  to
    logger.info("Received ping request");
*/


/* also in validators/index.ts 

change these
logger.info("Validating request body against schema" , {correlationId : req.headers["x-correlation-id"]});
logger.info("Request body is valid" , {correlationId : req.headers["x-correlation-id"]});

to
logger.info("Validating request body against schema" );
logger.info("Request body is valid");


*/
```

4) Testing it
```
// run the server: npm run dev

// output
{"level":"info","message":"Press CTRL+C to stop the server","timestamp":"06-04-2026 01:40:37","correlationId":"unknown-error-while-getting-correlation-id","data":{}}

// now send the request 
{"level":"info","message":"Validating request body against schema","timestamp":"06-04-2026 01:41:18","correlationId":"b1e00261-f731-40b1-b67d-4e485a510cdb","data":{}}
{"level":"info","message":"Request body is valid","timestamp":"06-04-2026 01:41:18","correlationId":"b1e00261-f731-40b1-b67d-4e485a510cdb","data":{}}
{"level":"info","message":"Received ping request","timestamp":"06-04-2026 01:41:18","correlationId":"b1e00261-f731-40b1-b67d-4e485a510cdb","data":{}}

```

5) Brief explanation
```
When a request comes from Postman, the attachCorrelationId middleware runs first and generates a unique UUID (correlation ID). It then creates an AsyncLocalStorage context using:

asyncLocalStorage.run(
  { correlationId },
  () => next()
);

Think of this as creating a small request-specific storage box containing the correlation ID. Every piece of code executed as part of that request (validators, controllers, services, repositories, database calls, and even code after await) automatically gets access to the same storage box. 
Later, when logger.info() is called, it internally executes getCorrelationId(), which reads the correlation ID from AsyncLocalStorage and adds it to the log. 
This is why you no longer need to manually pass correlationId through function parameters or include it in every log statement.
```


## ⬢ New Section : Storing Logs in a log file

1) We will then add the log to a particular file and add them in .gitignore
```ts
// in this file src/config/logger.config.ts
transports: [
    new winston.transports.Console(), // this will log to the console
    new winston.transports.File({ filename: 'logs/app.log' }) // this will log to a file named app.log in the logs directory
]
});

2) Logs being segregated day wise 
```ts

//1 : install winston daily rotate file
npm i winston-daily-rotate-file

//2: change to following code in this file src/config/logger.config.ts

// import this
import DailyRotateFile from "winston-daily-rotate-file";


// change transports to this
transports: [
    new winston.transports.Console(), // this will log to the console
    new DailyRotateFile({
        filename: 'logs/%DATE%.log', // this will create log files with the specified name pattern, where %DATE% will be replaced with the current date
        datePattern: 'YYYY-MM-DD', // this will rotate the log file daily, you can also specify other patterns like 'YYYY-MM-DD-HH' for hourly rotation
        maxFiles: '14d', // this will keep log files for 14 days and delete older files, you can also specify a number like '10' to keep only the latest 10 files
    })
]   
});


//3 : Test it again
// npm run dev
// and see the log files

```
