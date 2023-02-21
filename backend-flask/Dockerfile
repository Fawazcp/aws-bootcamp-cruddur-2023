#exampro bootcamp project docker

#FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask

FROM python:3.10-slim-buster

# inside container
#create a new folder inside the container
WORKDIR /backend-flask

# outside container --> inside container
# this contains the libraries to install to run the app
COPY requirements.txt requirements.txt

# Inside container
# install python libraries
RUN pip3 install -r requirements.txt

#outside container --> inside container
# .(this is period) it means everything in the current working directory
# first period= /backenf-flask (outside container)
# second period = /backenf-flask (inside container)
COPY . .

# set environment variables (inside container) and it will remain there when the container is running
ENV FLASK_ENV=development

EXPOSE ${PORT}
# python3 -m flask run --host=0.0.0.0 --port=4567 
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"] 