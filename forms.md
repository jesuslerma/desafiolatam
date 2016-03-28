# Formularios y almacenamiento en bd
## Lo que veremos en esta clase
  * routes get y post
  * formularios
  * models
  * migrations
  * guardar datos en postgres
Crear nuevo proyecto de rails, crear controlladores, crear modelos

## Antes de Empezar
  * Verificar que tengamos instalado postgres/psql

  ```bash
   $ /etc/init.d/postgresql status
   $ su - postgres
   $ psql
   #si no se saben el pass
   $ sudo passwd postgres
  ```
  * Instalar [postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)

# http y verbos
## Modelo Cliente Servidor
![alt text][cliente-serv]
[cliente-serv]: ./img/Cliente-Servidor.png
## Urls:

www.mydominio.com/photos

www.mydominio.com/photos/1

www.mydominio.com/photos/new

www.mydominio.com/photos/1/edit

## Tabla de verbos

![alt text][verbs]
[verbs]: ./img/verbs.png
# Crear nuevo proyecto de rails, crear controllers
```bash
$ rails new forms-web
$ rake routes #mostrara la ruta por defecto
$ rails g controller users
#eliminar contenido comentado de routes
```
```ruby
#config/routes.rb
Rails.application.routes.draw do
  resources :users, only: [:create]
end

#app/controllers/users_controller.rb
#agregar skip_before_filter :verify_authenticity_token
def create
	puts params
	head :no_content
end
```
```bash
$ rake routes
$ rails s
#probar rutas con postman
```

# Crear base de datos y usuario
```bash
  $ su - postgres
  $ psql
  CREATE USER desafio;
  ALTER ROLE desafio WITH password 'latam';
  ALTER ROLE desafio WITH superuser;
  CREATE DATABASE form_development;
```
#agregar pg al Gemfile
```ruby
#Gemfile
gem 'pg'
```
#modificar database.yml
```bash
$ rails g model user name:string age:integer email:string
$ rails c #va a mostrar un error
$ rake db:migrate
$ rails c
```
#crear usuarios desde consola
```ruby
User.all
User.create(email:"shuyojl@gmail.com", age: 21, name:"Jesús Lerma")
User.all
user = User.new(email:"shuyojl@gmail.com", age: 21, name:"Jesús Lerma")
user.save
User.all
params = {email:"shuyojl@gmail.com", age: 21, name:"Jesús Lerma"}
User.create params
user_params = User.new params
user_params.save
User.all
User.delete_all
User.all
#abrir postgress y consultar los registros
```

#Agregando codigo
```ruby
#forms-web/app/controllers/users_controller.rb
skip_before_filter :verify_authenticity_token

def create
  puts "I'm in"
  puts params
  user = User.new params
  user.save
  head :no_contenthead :no_content
end
#forbidden attributes
private
def user_params
	params.require(:user).permit(:age, :name, :email)
end
# hacer pruebas con postman
$ touch forms-web/app/views/users/new.html.erb
#config/routes.rb
Rails.application.routes.draw do
  resources :users, only: [:create, :new]
end
#app/controllers/users_controller.rb
def new
  render json: params
  @user = User.new
end
#forms-web/app/views/users/new.html.erb
#primero sin label luego con label
<%= form_for @user do |f| %>
	<%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.label :age %>
  <%= f.text_field :age %>
  <%= f.label :email %>
  <%= f.text_field :email %>
  <%= f.submit %>
<% end %>


#rails c
User.all
User.delete_all
```


# Mostrando Todos los Usuarios
```bash
$ touch app/views/users/index.html.erb
```
```ruby
#config/routes.rb
resources :users, only: [:new, :create, :index]
#app/controllers/users_controller.rb
def index
  @users = User.all
  @users.all.each do |user|
    puts user.attributes
    puts user.name
  end
end
def create
 redirect_to users_path
end
#app/views/users/index.html.erb
#mostrar diferencias entre
<% @users.all.each do |user| %>
  <%= user.name %> <%= user.age %> <%= user.email %><br>
<% end %>
y
<%= @users.all.each do |user| %>
<%= user.name %> <%= user.age %> <%= user.email %><br>
<% end %>
```

# Mostrando Errores
```ruby
#forms-web/app/controllers/users_controller.rb
#mostrar diff entre user y @user
def create
  @user = User.new user_params
  if @user.save
    redirect_to users_path, notice: 'Usuario registrado con exito'
  else
    render :new
  end
end
#app/models/user.rb
class User < ActiveRecord::Base
	validates :email, presence: true
end
#app/views/users/new.html.erb
<% if @user.errors.any? %>
  <div id="error_explanation">
    <ul>
    <% @user.errors.full_messages.each do |message| %>
      <li><%= message %></li>
    <% end %>
    </ul>
  </div>
<% end %>
<%= form_for @user do |f| %>
	<%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.label :age %>
  <%= f.text_field :age %>
  <%= f.label :email %>
  <%= f.text_field :email %>
  <%= f.submit %>
<% end %>

#app/views/users/index.html.erb
<p id="notice"><%= notice %></p>
<% @users.all.each do |user| %>
	<%= user.name %> <%= user.age %> <%= user.email %><br>
<% end %>
```
