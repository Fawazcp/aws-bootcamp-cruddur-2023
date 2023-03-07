# Week 3 — Decentralized Authentication

- Watched live stream video about [**Decentralized Authenication** on YouTube](https://www.youtube.com/live/9obl7rVgzJw?feature=share)

- Login to [AWS managemnet console](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1) 
- Open Gitpod and login to your AWS account using `aws cli`
-  Open **cognito** and Create user pool using AWS management console

**Install AWS Amplify**

cd frontend-react-js
```
npm i aws amplify --save
```

Add this in to **App.js**

```
import { Amplify } from 'aws-amplify';
```

```
Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
  "aws_cognito_identity_pool_id": process.env.REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```

- 👇Add this in your `docker-compose.yml` file

```
REACT_APP_AWS_PROJECT_REGION: ""
REACT_APP_AWS_COGNITO_REGION: ""
REACT_APP_AWS_USER_POOLS_ID: ""
REACT_APP_CLIENT_ID: ""
```

- Go to `HomeFeedPage.js` and add these codes

```
import { Auth } from 'aws-amplify';
```

<img width="590" alt="image" src="https://user-images.githubusercontent.com/111639918/223527302-e01a5912-5c0f-4816-9ba4-41a79a968ac1.png">


```
const checkAuth = async () => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};
```

- Update the below code in `Profileinfo.js` file

```
import { Auth } from 'aws-amplify';
```

```
const signOut = async () => {
  try {
      await Auth.signOut({ global: true });
      window.location.href = "/"
  } catch (error) {
      console.log('error signing out: ', error);
  }
}
```

### SignIn page

Update the code
```
import { Auth } from 'aws-amplify';
```


<img width="449" alt="image" src="https://user-images.githubusercontent.com/111639918/223538232-0e1486ca-3d9d-495e-851b-d0103490fbbd.png">

```
const onsubmit = async (event) => {
  setErrors('')
  event.preventDefault();
  try {
    Auth.signIn(email, password)
      .then(user => {
        localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
        window.location.href = "/"
      })
      .catch(err => { console.log('Error!', err) });
  } catch (error) {
    if (error.code == 'UserNotConfirmedException') {
      window.location.href = "/confirm"
    }
    setErrors(error.message)
  }
  return false
}

``` 

![image](https://user-images.githubusercontent.com/111639918/223541294-e7c2fbb0-85ea-4538-804e-fa71ac50f5cc.png)

![image](https://user-images.githubusercontent.com/111639918/223544952-380765c5-25f2-4a7d-99eb-2ea137850fc5.png)

  


