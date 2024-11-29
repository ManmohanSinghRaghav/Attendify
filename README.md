## Building and Running the Application

### Frontend

To build and run the React frontend application, use the following commands:

```sh
docker build -t my-react-app .
docker run -p 80:80 my-react-app
```

### Backend

To build and run the Face Recognition backend application, use the following commands:

```sh
docker build -t facereco-backend .
docker run -p 5000:5000 facereco-backend
```

## Or Using Docker Compose

To build and run the entire application using Docker Compose, use the following command:

```sh
docker-compose up --build
```