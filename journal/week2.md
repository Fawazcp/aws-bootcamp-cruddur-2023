# Week 2 — Distributed Tracing   

- Watched (Week 2) - Distributed Tracing live stream on [YouTube](https://www.youtube.com/live/2GD9xCzRId4?feature=share)

# 1️⃣Create Honeycomb account
To create honeycomb account watch the video posted by **Shala Warner** in her [YouTube channel]( https://youtu.be/7IwtVLfSD0o)

# Step 1

- Open Gitpod workspace and enter the below command

```
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

<img width="430" alt="image" src="https://user-images.githubusercontent.com/111639918/222540680-ccc5424f-b9e4-413d-ab88-d756809a4826.png">

- Add this env vars into docker-compose.yml file

```
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```

<img width="635" alt="image" src="https://user-images.githubusercontent.com/111639918/222539456-deb76670-8d14-4923-a117-c6f42d301d90.png">

Add this command in to requiremnt.txt file
```
    opentelemetry-sdk \
    opentelemetry-exporter-otlp-proto-http \
    opentelemetry-instrumentation-flask \
    opentelemetry-instrumentation-requests
```
```
pip install -r requirements.txt
```
![image](https://user-images.githubusercontent.com/111639918/222542566-2c8ec195-b24f-49cd-bfec-bae43d264242.png)


- **Copy and paste the below command in app.py file**

```
# app.py updates
    
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()

```


![image](https://user-images.githubusercontent.com/111639918/222543399-7e55b41d-aaa8-4506-b49d-1acf34818cc9.png)

- Add  a tracer in `home_activities.py`

```
from opentelemetry import trace
tracer = trace.get_tracer("home.Activities") # this will show up in attribute of field library
```

```
with tracer.start_as_current_span("home-activites-mock-data"):
    span = trace.get_current_span() # this will get whatever span it's in
    now = datetime.now(timezone.utc).astimezone()
    span.set_attribute("app.now", now.isoformat()) # this app.now attribute will show inside this span "home-activites-mock-data" , its data is the time now in ISO foramt.
```
    
    
    
![image](https://user-images.githubusercontent.com/111639918/222809320-82a419e2-cb06-443f-944e-02f6bc4171c0.png)

- **Docker compose UP**

When I do docker compose up I am not getting any trace in Honeycomb account instead getting an error says `Errno 111 connection refused`




<img width="608" alt="honeycomb" src="https://user-images.githubusercontent.com/111639918/222810247-3b9439af-e383-4f2b-b328-c827c51c667f.png">

Seems like issue with GitPod.  I searched about this error on chatgpt and google and got response says that maybe because of network issue. I also seeked help in Discord and there are bunch of people spend their time to help me but unfortunately no one coudn't find the solution. So I decided to move next challenge and try resolve this error after sometime.



![image](https://user-images.githubusercontent.com/111639918/222811114-ccf24f65-4465-4930-8f23-8807493ffe7d.png)

- I realized that I removed the dynamodb and postgresql scripts from `docker-compose.yml` file. After adding dynamodb and postgresql script in the docker file  I got the output



![image](https://user-images.githubusercontent.com/111639918/222814748-47328391-14f4-48b1-9de4-0936123bad94.png)


![image](https://user-images.githubusercontent.com/111639918/222816226-81cefdb1-17d3-4423-88f4-aac0e2e94c01.png)

![image](https://user-images.githubusercontent.com/111639918/222821504-77d2c27b-b8f7-4ddf-9463-ee2d32293d02.png)

# 2️⃣Instrument XRay

- Watched Instrument X-ray video posted on [YouTube](https://youtu.be/n2DTsuBrD_A) by Andrew Brown

**👇ADD this into `requirement.txt` file**

```
aws-xray-sdk
```

```
cd backend-flask 
pip install -r requirement.txt
```

**ADD this into app.py**
```
# X-RAY -------
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
```

```
# X-RAY -------
xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
```

<img width="841" alt="image" src="https://user-images.githubusercontent.com/111639918/222921459-b9ec25f2-2248-4608-86bb-3fcd6b092305.png">

- **Add aws/json/xray.json**

```
{
    "SamplingRule": {
        "RuleName": "Cruddur",
        "ResourceARN": "*",
        "Priority": 9000,
        "FixedRate": 0.1,
        "ReservoirSize": 5,
        "ServiceName": "backend-flask",
        "ServiceType": "*",
        "Host": "*",
        "HTTPMethod": "*",
        "URLPath": "*",
        "Version": 1
    }
  }
```

- Add this command. Make sure you are in to backend repo
```
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```   

 - To verify the group is created go to AWS management console--> Xray --> Groups

![image](https://user-images.githubusercontent.com/111639918/222951312-b2c2f0fe-de9a-4512-a720-2e2479fe9f64.png)
Run this command

```
cd ..
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
![image](https://user-images.githubusercontent.com/111639918/222924382-517d2260-641a-468f-aaf3-3c4707ed7bef.png)




![image](https://user-images.githubusercontent.com/111639918/222924448-f6cdd136-73e9-407d-b4af-d46e1b504e79.png)

### Install Daemon Service 

- **Add Deamon Service to `docker-compose.yml`**

```
 xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
  ```    
  <img width="579" alt="image" src="https://user-images.githubusercontent.com/111639918/222924845-5da76ec2-5a41-4405-88cc-555af2806bda.png">
  
  # Error
  
  - When I do docker-compose up the data is not getting traced. I am getting an error saying NoCredentialProviders


  ![image](https://user-images.githubusercontent.com/111639918/223091967-98409546-ea71-4639-8df0-8abd75d1fcd4.png)
  
  - I seeked help in Discord and one of the bootcamp participant noticed that I haven't set my aws credentials in my `docker-compose` file. After adding my credentials in my x-ray data traces are available 👇


  ![image](https://user-images.githubusercontent.com/111639918/223102521-89139b78-a6cc-4127-bf9f-ebb83b5b4162.png)
  
  ![image](https://user-images.githubusercontent.com/111639918/223105447-32509ce9-c6ff-4cab-92e1-069cd4817ff0.png)

  
  ![image](https://user-images.githubusercontent.com/111639918/223103539-b3b29175-a490-40f3-aef7-1799dc83871e.png)
  
 # 3️⃣ Cloudwatch Logs
  
  - To implement cloudwatch logs I watched video posted on [YouTube by Andrew Brown](https://youtu.be/ipdFizZjOF4)

To install **watchtower** add this command into requirements.txt

```
watchtower
```
cd backend-flask/

```
pip installl -r requirments.txt
```

Open **app.py** and the below command 

```
import watchtower
import logging
from time import strftime
```

```
# Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("some message")
```

<img width="752" alt="image" src="https://user-images.githubusercontent.com/111639918/223186757-97c5cc04-cda7-4c88-ad8b-6ea535bedf9a.png">

- Set the env var for docker-compose.yml

```
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```      

<img width="592" alt="image" src="https://user-images.githubusercontent.com/111639918/223191374-889707f8-dcbe-43d0-af74-aa7b45e1c40c.png">

- docker-compose up
- Open backend port and refresh
-  Go to cloudwatch log group

![image](https://user-images.githubusercontent.com/111639918/223199749-24d96055-250f-400c-b203-d314ed670b0d.png)

![image](https://user-images.githubusercontent.com/111639918/223200085-a7f83769-f073-43d7-a00c-435f3a09d91c.png)

 # 4️⃣Rollbar

- Create a rollbar account. To create a rollbar account watch video posted on [YouTube by **Shala Warner**](https://youtu.be/Lpm6oAP3Fb0) or 
-  https://rollbar.com/
-  Watch  video posted on YouTube by Andrew Brown about [rollbar](https://youtu.be/xMBDAb5SEU4)

- Add these into requirements.txt file

```
blinker
rollbar
```

cd backend-flask/
```
pip install -r requirements.txt
```

- Set access token

```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```
- To confirm the rollbar token exported

```
env | grep ROLLBAR
```

- Add these in to app.py

```
import os
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```

```
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
 ```
 
 ```
 @app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```

- docker-compose up

![image](https://user-images.githubusercontent.com/111639918/223208808-dd02972b-56ee-434a-9fe9-1b25438612f4.png)

- Add rollbar access token into `docker-compose.yml` file

<img width="581" alt="image" src="https://user-images.githubusercontent.com/111639918/223211604-28c8d7b8-e869-45c4-a9d8-19fa353c2f3a.png">


    <img width="955" alt="image" src="https://user-images.githubusercontent.com/111639918/223213199-852dc587-45fd-4925-b5e7-7ff82ead4225.png">

![image](https://user-images.githubusercontent.com/111639918/223213957-db989eaa-8232-4bf0-b8a2-d728fb42e603.png)


### Summary

| Homework      | Completed     | Not Completed  |
| ------------- |:-------------:| -----:|
| Watch Week 2 Live-Stream Video   | ✔ |  |
|	Watch Chirag Week 2 - Spending Considerations | ✔     |    |
| Watched Ashish's Week 2 - Observability Security Considerations | ✔      |   |
|Instrument Honeycomb with OTEL|✔      |   |
|Instrument AWS X-Ray|✔      |   |
|Instrument AWS X-Ray Subsegments|    ✔  |   |
| Configure custom logger to send to CloudWatch Logs  | ✔   |   |
|Integrate Rollbar and capture and error |✔      |   |



   



