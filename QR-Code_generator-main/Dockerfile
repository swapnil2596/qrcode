FROM python:3.6

COPY . /app

WORKDIR /app

RUN pip3 install -r requirement.txt

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "iotbackend.wsgi:application"]

# # "--settings=your_project_name.settings"


# FROM python:3.6

# WORKDIR /app

# COPY requirement.txt requirement.txt
# RUN pip3 install -r requirement.txt

# COPY . .

# RUN mkdir Logs

# CMD [ "python3", "manage.py" , "runserver",  "0.0.0.0:8000"]
# EXPOSE 8000
