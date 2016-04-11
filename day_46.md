# Acuerdos
1. Las lineas que empiezan con $ se deben de ejecutar en terminal
2. Para indicar la ruta del archivo que se va a modificar se escribe un hashtag previo y el path es partiendo el root del proyecto. Por ejemplo: #app/models/user.rb
3. Para indicar codigo en sql se pone al inicio de la linea sql>
4. Para indicar codigo en rails console se pone el comentario ejecutamos rails c
5. Yo voy a estar trabajando siempre con la misma base de datos form_development para conectarme a ella uso el siguiente comando
```bash
$ psql -h localhost form_development  desafio  -W.
```
Pueden utilizar la misma bd o crear una nueva previamente a cada ejercicio. Siempre vamos a conectarnos a psql.

El contenido mostrado a continuación es de ayuda para el material didáctico de desafio latam 2016. Este archivo hace referencia a la clase
46: Logica de negocio

# Ejercicio 1 Lógica de negocios
En terminal corremos lo siguiente:
```bash
$ rails new zoo
$ cd zoo
$ rails g model animal_type name:string
$ rails g model animal name:string animal_type_id:integer:index
$ rake db:migrate
```
### Taller en clases:
Para ir prácticando creamos un proyecto nuevo con la configuracion que tenemos configurada por defecto en database.yml. Las instrucciones de este ejercicio vienen en la presentacion 46_logica_de_negocios en la pagina 23.
```ruby
#app/models/animal_type.rb
...
has_many :animals
validates :name, presence: true
...
#app/models/animal.rb
...
belongs_to :animal_type
...
#ejecutamos rails c
animal_type = AnimalType.new name:"chango"
animal_type.valid?
animal_type.name = nil
animal_type.valid?
animal_type.errors
```

### Ejercicio en clases 2
Para seguir prácticando creamos un proyecto nuevo con la configuracion que tenemos configurada por defecto en database.yml. Las instrucciones de este ejercicio vienen en la presentacion 46_logica_de_negocios en la pagina 23.
```bash
$ rails new blog_demo
$ cd blog_demo
$ rails g model post title:string
$ rails g model user name:string
$ rails g migration AddUserIdToPost user_id:integer:index
$ rails g model comment content:text user_id:integer:index post_id:integer:index
$ rake db:migrate
```

```ruby
#app/models/post.rb
...
belongs_to :user
has_many :comments
validates :user_id, presence: true
before_validation do
  default_user = User.first
  self.user_id = default_user.id if self.user_id.blank?
end
...
#app/models/user.rb
...
has_many :posts
has_many :comments
before_destroy do
  default_user = User.first
  Post.where(user_id:id).update_all(user_id: default_user.id)
end
#app/models/comment.rb
...
belongs_to :user
belongs_to :post
validates :user_id, presence: true
...
```
## Validaciones Personalizadas (custom validations)
El esqueleto base para crear una validación personalizada es:
```ruby
validate :custom_method_validation
def custom_method_validation
  if condition_to_break
    #do something like add error message
  end
end
```

Para practicar podemos validar en el modelo user del proyecto blog_demo:

```ruby
#app/models/comment.rb
validate :post_has_valid_date?

private
def post_has_valid_date?
  limit_day = DateTime.now - 5.days
  if self.post.created_at < limit_day
    #explain diference btwn base and created_at symbols
    #errors.add(:base, "You can't add comments to posts with 5 or more days of created age")
    errors.add(:created_at, "You can't add comments to posts with 5 or more days of creation date")
  end
end
```

### Scopes
Ahora regresamos al proyecto zoo que creamos en el ejercicio anterior.
```ruby
#app/models/animal.rb
...
def self.monkeys
  joins(:animal_type).where("animal_type.name = 'chango'")
end
scope :monkeys, joins(:animal_type).where("animal_types.name = 'chango'")
scope :monkeys, lambda {joins(:animal_type).where("animal_types.name = 'chango'")}
scope :monkeys, -> {joins(:animal_type).where("animal_types.name = 'chango'")}

scope :last_five, last(5)
scope :last_five, lambda {last(5)}
scope :last_five, -> {last(5)}

scope :by_animal_type, lambda{ |type| joins(:animal_type).where("animal_type.name = #{type}") }
scope :by_animal_type, ->(type) {joins(:animal_type).where("animal_type.name = #{type}") }
...
```
