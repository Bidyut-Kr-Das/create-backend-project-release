## Package: Create-backend-project

### Description: Creates entire express backend project with all the necessary files and folders that follows industry standard template.

### Usage:

```bash
npx create-backend-app
```
```bash
cd <project_name>
npm install
```
<br>

> write your database name in the `database/index.js` 
```javascript
const DBName = ""; //replace your database name
```
<br>

> Add your `mongoDB connection string`, `password` and `CORS origin` in the `.env` file
```env
  MONGO_URI=mongodb+srv://USERNAME:<db_password>@cluster-test.dyzrsx0.mongodb.net/
  MONGO_PASS=abcde12345
  CORS_ORIGIN=http://localhost:3000
```



##### Folder structure:

```
project-name
├── src
│   ├── app.js
│   ├── server.js
│   ├── constants.js
│   ├── models
│   ├── routes
│   ├── utils
│   │   ├── catchAsyncError.js
│   │   └── appError.js
│   ├── cors
│   │   └── index.js
│   ├── database
│   │   └── index.js
│   ├── middlewares
│   │   └── error.middleware.js
│   └── controllers
│       └── error.controller.js
├── .gitignore
├── .env
└── package.json
```

<br>

> Comoponents

1. #### Error handling middleware

```javascript
const handleError = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || `error`;
  if (process.env.NODE_ENV === `development`) {
    return sendDevelopmentError(err, res);
  }
  let error = { ...err };
  error.message = err.message;
  // console.log(error);
  // console.log(err.message);
  if (err.code === 11000) error = duplicationError(error);
  return sendProductionError(error, res);
};
```

- ##### Usage:
  This middleware will be used to handle all the errors in the application. It will send a detailed error message in the development environment and will send a user-friendly message in the production environment.

2. #### CatchAsyncError

```javascript
const catchAsyncError = (cb) => {
  return (req, res, next) => {
    cb(req, res, next).catch((error) => {
      next(error);
    });
  };
};
export default catchAsyncError;
```

- ##### Usage:
  1. No need to write `try-catch` block in the controller function.
      <br>
  2. Wrap all your controller function with this `catchAsyncError(controller_function)` function to handle all the errors in the application.
      <br>
  3. Import the `class AppError` from `#utils/appError.js` and Just `throw new AppError(message, statusCode)` the error will be handled automatically by the AppError class.
  > Example:

  ```javascript
  import catchAsyncError from '#utils/catchAsyncError.js';
  import AppError from '#utils/appError.js';

  export const getOne = catchAsyncError(async (req, res, next) => {
    const user = await User.findById(req.params.id);
      if (!user) {
          throw new AppError(`No user found with that ID`, 404);
      }
      res.status(200).json({
          status: `success`,
          data: {
              user
          }
      });
  });
  ```
  <br>

3. #### MongoDb Connection

```javascript
import mongoose from "mongoose";

const DBName = ""; //replace your database name

const connectDB = async () => {
  const DBUrl = process.env.MONGO_URI.replace(
    `<db_password>`, //make sure names in env are correct and the password part in the uri is same like this
    process.env.MONGO_PASS
  );
  const response = await mongoose.connect(`${DBUrl}/${DBName}`);
};
export default connectDB;
```

- ##### Usage:

  1. Replace the `DBName` with your database name.
     <br>
  2. In `.env` file add the mongoDB connection string into the `MONGO_URI`.
     <br>
  3. In `.env` file add your mongoDb into the `MONGO_PASS`.

  > Example:

  ```env
  MONGO_URI=mongodb+srv://USERNAME:<db_password>@cluster-test.dyzrsx0.mongodb.net/
  MONGO_PASS=abcde12345
  ```

4. #### CORS

```javascript
import cors from "cors";

const corsOptions = {
  origin: process.env.CORS_ORIGIN || "http://localhost:8003",
  optionsSuccessStatus: 200,
  credentials: true,
};

export default cors(corsOptions);
```

- ##### Usage:

  1. Add the origin of the frontend in the `.env` file into `CORS_ORIGIN`.
     <br>

  > Example:

  ```env
  CORS_ORIGIN=http://localhost:3000
  ```

> Please raise an issue if you find any bugs or need any new features. I will try to resolve it as soon as possible.

Thank you for using this package.
