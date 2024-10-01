#Commands
- python -m venv env
- .\env\Scripts\activate
- pip install django
- python.exe -m pip install --upgrade pip
- django-admin startproject server
- pip install mysqlclient
- python manage.py startapp users
- PocDevWeb/server/server/settings.py -> 
  ```
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'marcalmo',
          'USER': 'root',
          'PASSWORD': 'IBD.dcc.ufmb',
          'HOST': 'localhost',
          'PORT': '3306',
      }
  }
  ```
- PocDevWeb/server/users/models.py -> 
  ```
  class User(models.Model):
      name = models.CharField(max_length=100)
      email = models.EmailField(unique=True)
      phone = models.CharField(max_length=15)

      def __str__(self):
          return self.name
  ```
- pip install djangorestframework
- Create: PocDevWeb/server/users/serializers.py ->
```
  from rest_framework import serializers
  from .models import User

  class UserSerializer(serializers.ModelSerializer):
      class Meta:
          model = User
          fields = ['id', 'name', 'email', 'phone']
```
- PocDevWeb/server/users/views.py -> 
```
  from rest_framework import status
  from rest_framework.response import Response
  from rest_framework.decorators import api_view
  from .models import User
  from .serializers import UserSerializer

  @api_view(['GET', 'POST'])
  def user_list(request):
      if request.method == 'GET':
          users = User.objects.all()
          serializer = UserSerializer(users, many=True)
          return Response(serializer.data)

      if request.method == 'POST':
          serializer = UserSerializer(data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data, status=status.HTTP_201_CREATED)
          return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

  @api_view(['GET', 'PUT', 'DELETE'])
  def user_detail(request, pk):
      try:
          user = User.objects.get(pk=pk)
      except User.DoesNotExist:
          return Response(status=status.HTTP_404_NOT_FOUND)

      if request.method == 'GET':
          serializer = UserSerializer(user)
          return Response(serializer.data)

      if request.method == 'PUT':
          serializer = UserSerializer(user, data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data)
          return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

      if request.method == 'DELETE':
          user.delete()
          return Response(status=status.HTTP_204_NO_CONTENT)
```
- Create: PocDevWeb/server/users/urls.py ->
```
  from django.urls import path
  from . import views

  urlpatterns = [
      path('users/', views.user_list),
      path('users/<int:pk>/', views.user_detail),
  ]
```
- PocDevWeb/server/server/urls.py ->
```
  from django.contrib import admin
  from django.urls import path, include

  urlpatterns = [
      path('admin/', admin.site.urls),
      path('api/', include('users.urls')),
  ]
```
- PocDevWeb/server/server/settings.py
```
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'users', # add this line
  ]
```
- python manage.py makemigrations
- python manage.py migrate
- python manage.py runserver
- Routes (Testing in Postman for example)
  - GET http://127.0.0.1:8000/api/users/
  - GET http://127.0.0.1:8000/api/users/<user_id>/
  - POST http://127.0.0.1:8000/api/users/
  - PUT http://127.0.0.1:8000/api/users/<user_id>/
  - DELETE http://127.0.0.1:8000/api/users/<user_id>/

## Finish Backend ##
## Start Frontend ##
- node -v -> v20.17.0
- npm -v -> 10.8.2
- Obs: maybe you need create npm folder inside C:\Users\lnv\AppData\Roaming manually
- npx create-react-app frontend -> y
- npm install axios
- PocDevWeb/frontend/src/services/UserService.js
```
import axios from 'axios';

const API_URL = 'http://127.0.0.1:8000/api/users/';  // Your Django API base URL

const getUsers = () => {
    return axios.get(API_URL);
};

const getUser = (id) => {
    return axios.get(`${API_URL}${id}/`);
};

const createUser = (data) => {
    return axios.post(API_URL, data);
};

const updateUser = (id, data) => {
    return axios.put(`${API_URL}${id}/`, data);
};

const deleteUser = (id) => {
    return axios.delete(`${API_URL}${id}/`);
};

export default { getUsers, getUser, createUser, updateUser, deleteUser };
```


- PocDevWeb/frontend/src/components/UserList.js
import React, { useEffect, useState } from 'react';
import UserService from '../services/UserService';

function UserList() {
    const [users, setUsers] = useState([]);

    useEffect(() => {
        fetchUsers();
    }, []);

    const fetchUsers = async () => {
        try {
            const response = await UserService.getUsers();
            setUsers(response.data);
        } catch (error) {
            console.error("There was an error fetching the users!", error);
        }
    };

    return (
        <div>
            <h2>User List</h2>
            <ul>
                {users.map((user) => (
                    <li key={user.id}>
                        {user.name} - {user.email} - {user.phone}
                    </li>
                ))}
            </ul>
        </div>
    );
}

export default UserList;

- PocDevWeb/frontend/src/component/AddUser.js
```
    import React, { useState } from 'react';
    import UserService from '../services/UserService';

    function AddUser() {
        const [name, setName] = useState('');
        const [email, setEmail] = useState('');
        const [phone, setPhone] = useState('');

        const handleSubmit = async (e) => {
            e.preventDefault();
            const user = { name, email, phone };
            try {
                await UserService.createUser(user);
                alert('User added successfully');
            } catch (error) {
                console.error('There was an error creating the user!', error);
            }
        };

        return (
            <div>
                <h2>Add User</h2>
                <form onSubmit={handleSubmit}>
                    <div>
                        <label>Name:</label>
                        <input
                            type="text"
                            value={name}
                            onChange={(e) => setName(e.target.value)}
                        />
                    </div>
                    <div>
                        <label>Email:</label>
                        <input
                            type="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                        />
                    </div>
                    <div>
                        <label>Phone:</label>
                        <input
                            type="text"
                            value={phone}
                            onChange={(e) => setPhone(e.target.value)}
                        />
                    </div>
                    <button type="submit">Add User</button>
                </form>
            </div>
        );
    }

    export default AddUser;
```

# Permission FrontEnd access BackEnd
- Go to Server in env enviroment
- pip install django-cors-headers
    ```
    INSTALLED_APPS = [
        ...
        'corsheaders',
        ...
    ]

    MIDDLEWARE = [
        ...
        'corsheaders.middleware.CorsMiddleware',
        'django.middleware.common.CommonMiddleware',
        ...
    ]

    CORS_ALLOWED_ORIGINS = [
        'http://localhost:3000',  # React app running locally
    ]
    ```

# Finally execute the Server and Client
- Terminal 01 in server with env activivate: python manage.py runserver
- Terminal 02 in backend: npm start
- Add an user and confirm in MySQL database