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

<img width="512" alt="image" src="https://user-images.githubusercontent.com/111639918/223691905-230f27c2-e6e9-4c46-b41e-0fec09beb3c3.png">

- Run this command to force change password for aws cognito https://stackoverflow.com/questions/40287012/how-to-change-user-status-force-change-password

```
aws cognito-idp admin-set-user-password \
  --user-pool-id <your-user-pool-id> \
  --username <username> \
  --password <password> \
  --permanent
```


  ![image](https://user-images.githubusercontent.com/111639918/223794826-97d4c48d-17cd-4aab-bb4c-70394987582d.png)

- Go to cognito Edit user and add your name

<img width="199" alt="image" src="https://user-images.githubusercontent.com/111639918/223804716-e375941e-997f-4bc2-9154-bb0acc583073.png">


- Update SighUp page

```
# enter this code in `SignUpPage.js`

import { Auth } from 'aws-amplify';
```

```
  const onsubmit = async (event) => {
    event.preventDefault();
    setErrors('')
    try {
        const { user } = await Auth.signUp({
          username: email,
          password: password,
          attributes: {
              name: name,
              email: email,
              preferred_username: username,
          },
          autoSignIn: { // optional - enables auto sign in after user is confirmed
              enabled: true,
          }
        });
        console.log(user);
        window.location.href = `/confirm?email=${email}`
    } catch (error) {
        console.log(error);
        setErrors(error.message)
    }
    return false
  }
  ```
  
  
  - Go to `ConfirmationPage.js` and update the below code

```
import { Auth } from 'aws-amplify';
```

```
const resend_code = async (event) => {
  setErrors('')
  try {
    await Auth.resendSignUp(email);
    console.log('code resent successfully');
    setCodeSent(true)
  } catch (err) {
    // does not return a code
    // does cognito always return english
    // for this to be an okay match?
    console.log(err)
    if (err.message == 'Username cannot be empty'){
      setErrors("You need to provide an email in order to send Resend Activiation Code")   
    } else if (err.message == "Username/client id combination not found."){
      setErrors("Email is invalid or cannot be found.")   
    }
  }
}
```

```
const onsubmit = async (event) => {
  event.preventDefault();
  setErrors('')
  try {
    await Auth.confirmSignUp(email, code);
    window.location.href = "/"
  } catch (error) {
    setErrors(error.message)
  }
  return false
}
```

- Throwing some error while **SignUp**


 ![image](https://user-images.githubusercontent.com/111639918/223824621-8ea24ac8-5d19-4229-bd22-7cb7b2df735e.png)
 
 - Recreated `user-pool` as per the instruction and the issue got fixed

![image](https://user-images.githubusercontent.com/111639918/223827821-f32d5d74-3415-43e7-974b-0aba432b2fc4.png)


- Confirm in **Cognito** the user has been created

<img width="714" alt="image" src="https://user-images.githubusercontent.com/111639918/223831159-9e848588-1e08-4bd2-bb33-8798a0892029.png">

### Password Recovery

- Go to `RecoverPage.js` and update the below code

```
import { Auth } from 'aws-amplify';
```

```
const onsubmit_send_code = async (event) => {
  event.preventDefault();
  setErrors('')
  Auth.forgotPassword(username)
  .then((data) => setFormState('confirm_code') )
  .catch((err) => setErrors(err.message) );
  return false
}
```

```
const onsubmit_confirm_code = async (event) => {
  event.preventDefault();
  setErrors('')
  if (password == passwordAgain){
    Auth.forgotPasswordSubmit(username, code, password)
    .then((data) => setFormState('success'))
    .catch((err) => setCognitoErrors(err.message) );
  } else {
    setErrors('Passwords do not match')
  }
  return false
}
```


![image](https://user-images.githubusercontent.com/111639918/223835172-8fa90733-0710-45fe-8c99-30c2214c9086.png)





  
