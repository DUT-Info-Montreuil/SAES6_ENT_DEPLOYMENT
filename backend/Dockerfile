FROM python:3.12-slim

WORKDIR /app

COPY ./SAES6_ENT_BACKEND/requirements.txt .
RUN pip install -r ./requirements.txt
RUN pip install gunicorn psycopg2-binary

COPY ./SAES6_ENT_BACKEND/. .

ENV FLASK_DEBUG=False
ENV WORKERS=1

EXPOSE ${SERVER_PORT}

CMD ["sh", "-c", "gunicorn -b 0.0.0.0:5000 -w ${WORKERS} 'app:create_app()'"]