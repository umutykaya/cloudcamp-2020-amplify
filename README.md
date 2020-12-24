## Fullstack Serverless

In this tutorial we will build fullstack serverless app that has REST API and React UI components.

* ``` npx create-react-app hello-react``` 
* ```cd hello-react```
* ```yarn add aws-amplify```
* ``amplify init`` 
* ``amplify add api``
  * **Service:** REST
  * **Project:** mypythonapi
  * **Path:** /greeting
  * **Lambda source:** Create new lambda function
  * **Friendly name:** mypythonfunction
  * **Function name:** mypythonfunction
  * **Runtime:** Python
  * **Others:** No
* `amplify push --y`
* `amplify hosting add` 
  * **Select the plugin module to execute:** Amazon CloudFront and S3
  * **Select the environment setup:** DEV (S3 only with HTTP)
  * **Hosting bucket name:** helloreact-20201222120745-hostingbucket
  * **Index doc for the website:** ``index.html`` 
  * **Error doc for the website:**  ``index.html``
* `amplify publish` 
  * http://helloreact-20201222120745-hostingbucket-dev.s3-website-eu-west-1.amazonaws.com/

#### Issue Patch

Amplify provide ``amplify push --y`` command to provision resources as cloudformation template. It only provides ``python >= 3.8`` version. Issue is still in progress on github on late December 2020.  Below is a workaround on ``Amazon Linux 2``  OS.

https://github.com/aws-amplify/amplify-console/issues/595

```
amazon-linux-extras install python3.8
ln -fs /usr/bin/python3.8 /usr/bin/python3
pip3.8 install pipenv
amplifyPush --simple
```

### 

### Backend API

The Lambda function template generated by ``amplify add api`` command. So, there should be edited on your IDE (Cloud9, VSCode, Atom etc.)

You can find `index.py` file in the `/hello-react/amplify/backend/function/mypythonfunction/src/` path.

```python
import json

def handler(event, context):
  body = {
    "message": "Hello from Lambda !"
  }
  
  response = {
    "statusCode": 200,
    "body": json.dumps(body),
    "headers": {
      "Content-Type": "application/json",
      "Access-Control-Allow-Origin": "*"
    }
  }
  
  return response
```

Then, we are ready to push our backend api to `AWS Lambda` and `API Gateway` via `amplify push --y ` command.

* API gateway endpoint: https://pk855su8od.execute-api.eu-west-1.amazonaws.com/dev
* You can double check properties from `aws-export.js` file.

```javascript
// WARNING: DO NOT EDIT. This file is automatically generated by AWS Amplify. It will be overwritten.

const awsmobile = {
    "aws_project_region": "eu-west-1",
    "aws_cloud_logic_custom": [
        {
            "name": "mypythonapi",
            "endpoint": "https://pk855su8od.execute-api.eu-west-1.amazonaws.com/dev",
            "region": "eu-west-1"
        }
    ]
};


export default awsmobile;
```



### Frontend 

Following block is ``index.js`` 

* ``npm install aws-amplify``

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

import Amplify from 'aws-amplify'; // load library dependencies
import awsconfig from './aws-exports'; // load backend exports
Amplify.configure(awsconfig); // configure amplify to use credentials

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

Following block is ``app.js`` 

```javascript
import React, {useState, useEffect } from 'react';
import logo from './logo.svg';
import './App.css';

import { API } from 'aws-amplify' // import amplify dependencies

function App() {
  const [greeting, setGreeting] = useState(null) // construct greeting and setGreeting 
  async function fetchGreeting() { // declare an asynch function called fetchGreeting
    const apiData = await API.get('mypythonapi', '/greeting') // Get API by using amplify library
    setGreeting(apiData.message) // Set message property to the response of fetchGreeting() 
  }
  useEffect(() => { // useEffect hook for async func. transition 
    fetchGreeting()
  }, [])
  
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
       <h1>{greeting}</h1> <! -- Call greeting constructor --> 
      </header>
    </div>
  );
}

export default App;

```



### References:

* https://docs.amplify.aws/
* https://github.com/dabit3