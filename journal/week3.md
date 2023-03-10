# Week 3 — Decentralized Authentication

- Watched live stream video about [**Decentralized Authenication** on YouTube](https://www.youtube.com/live/9obl7rVgzJw?feature=share)

- Login to [AWS managemnet console](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1) 
- Open Gitpod and login to your AWS account using **`aws cli`**
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

- 👇Add this in your **`docker-compose.yml`** file

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

- Watched video posted on YouTube uploaded in [**ExamPro** channel](https://youtu.be/T4X4yIzejTc)

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


### Update SighUp page

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
  
  
  - Go to **`ConfirmationPage.js`** and update the below code

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

- Go to **`RecoverPage.js`** and update the below code

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

### Watch Week 3 [Congito JWT Server side Verify video on YouTube](Week 3 Congito JWT Server side Verify)

📌**What is JWT token?**

- **JSON Web Token is an open industry standard used to share information between two entities, usually a client (like your app's frontend) and a server (your app's backend). They contain JSON objects which have the information that needs to be shared.**

- Update headers in **`HomeFeedPage.js`**

```
      headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
```

- goto **`app.py`** and add the below code

```
import sys
```

### Error

![image](https://user-images.githubusercontent.com/111639918/224355880-dd429eb4-5684-422b-81fd-762efb26b52f.png)

- Upadate the `CORS` in the `app.py` to resolve this issue

```
cors = CORS(
  app, 
  resources={r"/api/*": {"origins": origins}},
  headers=['Content-Type', 'Authorization'], 
  expose_headers='Authorization',
  methods="OPTIONS,GET,HEAD,POST"
)
```

- After updating the **`CORS`** my homepage is visible

![image](https://user-images.githubusercontent.com/111639918/224357144-9ac99bcd-e8ba-4c9a-bb55-94063e0fea64.png)


- Update the below code in the `docker-compose.yml` file

```
AWS_COGNITO_USER_POOL_ID: "user-pool-id"
AWS_COGNITO_USER_POOL_CLIENT_ID: "client-id"      
```      

- Create a new folder named `lib` inside `backend-flask`
- Create a file named `cognito_jwt_token.py` inside the `lib` folder
- add the below code into this file

``` 
import time
import requests
from jose import jwk, jwt
from jose.exceptions import JOSEError
from jose.utils import base64url_decode

class FlaskAWSCognitoError(Exception):
  pass

class TokenVerifyError(Exception):
  pass

class CognitoJwtToken:
    def __init__(self, user_pool_id, user_pool_client_id, region, request_client=None):
        self.region = region
        if not self.region:
            raise FlaskAWSCognitoError("No AWS region provided")
        self.user_pool_id = user_pool_id
        self.user_pool_client_id = user_pool_client_id
        self.claims = None
        if not request_client:
            self.request_client = requests.get
        else:
            self.request_client = request_client
        self._load_jwk_keys()

    def _load_jwk_keys(self):
        keys_url = f"https://cognito-idp.{self.region}.amazonaws.com/{self.user_pool_id}/.well-known/jwks.json"
        try:
            response = self.request_client(keys_url)
            self.jwk_keys = response.json()["keys"]
        except requests.exceptions.RequestException as e:
            raise FlaskAWSCognitoError(str(e)) from e

    @staticmethod
    def _extract_headers(token):
        try:
            headers = jwt.get_unverified_headers(token)
            return headers
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    def _find_pkey(self, headers):
        kid = headers["kid"]
        # search for the kid in the downloaded public keys
        key_index = -1
        for i in range(len(self.jwk_keys)):
            if kid == self.jwk_keys[i]["kid"]:
                key_index = i
                break
        if key_index == -1:
            raise TokenVerifyError("Public key not found in jwks.json")
        return self.jwk_keys[key_index]

    @staticmethod
    def _verify_signature(token, pkey_data):
        try:
            # construct the public key
            public_key = jwk.construct(pkey_data)
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e
        # get the last two sections of the token,
        # message and signature (encoded in base64)
        message, encoded_signature = str(token).rsplit(".", 1)
        # decode the signature
        decoded_signature = base64url_decode(encoded_signature.encode("utf-8"))
        # verify the signature
        if not public_key.verify(message.encode("utf8"), decoded_signature):
            raise TokenVerifyError("Signature verification failed")

    @staticmethod
    def _extract_claims(token):
        try:
            claims = jwt.get_unverified_claims(token)
            return claims
        except JOSEError as e:
            raise TokenVerifyError(str(e)) from e

    @staticmethod
    def _check_expiration(claims, current_time):
        if not current_time:
            current_time = time.time()
        if current_time > claims["exp"]:
            raise TokenVerifyError("Token is expired")  # probably another exception

    def _check_audience(self, claims):
        # and the Audience  (use claims['client_id'] if verifying an access token)
        audience = claims["aud"] if "aud" in claims else claims["client_id"]
        if audience != self.user_pool_client_id:
            raise TokenVerifyError("Token was not issued for this audience")

    def verify(self, token, current_time=None):
        """ https://github.com/awslabs/aws-support-tools/blob/master/Cognito/decode-verify-jwt/decode-verify-jwt.py """
        if not token:
            raise TokenVerifyError("No token provided")

        headers = self._extract_headers(token)
        pkey_data = self._find_pkey(headers)
        self._verify_signature(token, pkey_data)

        claims = self._extract_claims(token)
        self._check_expiration(claims, current_time)
        self._check_audience(claims)

        self.claims = claims
        return claim
   ```     
        
  - Go to **`app.py`** and update the below code

```
from lib.cognito_jwt_token import CognitoJwtToken
```

```
cognito_jwt_token = CognitoJwtToken(
  user_pool_id=os.getenv("AWS_COGNITO_USER_POOL_ID"),
  user_pool_client_id=os.getenv("AWS_COGNITO_USER_POOL_CLIENT_ID"),
  region=os.getenv("AWS_DEFAULT_REGION")
)
```

```
@app.route("/api/activities/home", methods=['GET'])
@xray_recorder.capture('activities_home')
def data_home():
  access_token = CognitoJwtToken.extract_access_token(request.headers)
  try:
    claims = cognito_jwt_token.token_service.verify(access_token)
  except TokenVerifyError as e:
      _ = request.data
      abort(make_response(jsonify(message=str(e)), 401))

  app.logger.debug('claims')
  app.logger.debug(claims)
  data = HomeActivities.run()
  return data, 200
```  

- Add the bleow code in to **`cognito_jwt_token.py`** file

```
@classmethod
def extract_access_token(request_headers):
    access_token = None
    auth_header = request_headers.get("Authorization")
    if auth_header and " " in auth_header:
        _, access_token = auth_header.split()
    return access_token
 ```
 
 ```
 HTTP_HEADER = "Authorization"
 ```
 
 
 - Add the below code in to **`home_activities.py`** file

```
    if cognito_user_id != None: 
      extra_crud = {
      'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
      'handle':  'Lore',
      'message': 'This one is added in JWT Week 3 class ',
      'created_at': (now - timedelta(hours=1)).isoformat(),
      'expires_at': (now + timedelta(hours=12)).isoformat(),
      'likes': 1042,
      'replies': []
    }
      results.insert(0,extra_crud)
  ```    
 
<img width="658" alt="image" src="https://user-images.githubusercontent.com/111639918/224413520-56fea254-e746-4638-aacb-2be563de78e4.png">

- Add the below code in **`ProfileInfo.js`** page

```
 localStorage.removeItem("access_token")
```

<img width="340" alt="image" src="https://user-images.githubusercontent.com/111639918/224415023-70ce1aea-ef24-4152-aa85-23db306f8003.png">

Sign-out from the cruddur app and we cannot see the latest message we added

![image](https://user-images.githubusercontent.com/111639918/224416345-4f47efb5-5951-4557-a169-8c2e3fe8d4dd.png)



#### Watch Week 3 -[ Exploring JWTs](https://youtu.be/nJjbI4BbasU)
