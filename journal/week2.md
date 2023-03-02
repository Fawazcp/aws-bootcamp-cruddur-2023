# Week 2 — Distributed Tracing   

- Watched (Week 2) - Distributed Tracing live stream on [YouTube](https://www.youtube.com/live/2GD9xCzRId4?feature=share)

# Create Honeycomb account
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


