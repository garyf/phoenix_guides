Example: Create a model with custom primary key `name:string` instead of autogenerated `id:integer`

> This functionality could be very useful in a variety of situations especially when adopting Phoenix within existing projects/organizations or databases/requirements that can not be changed and following proper Ecto way is not possible. It is important to remember that while Ecto gives us flexibility to do it, it's not necessarily the natural path Ecto wants us to take. 

> IMPORTANT NOTE: this example was chosen for simplicity.  It is not advisable to use something that is likely to change in the future as primary key.  

Create project named chat:

```console
$mix phoenix.new chat
$cd chat
$mix ecto.create
```

To create User model run: 
```console
$mix phoenix.gen.json User users name:string age:integer
```

We need these 3 simple changes in the following generated files:

**1. models/user.ex:**

Add `@primary_key {:name, :string, []}` right above `schema "users" do` and remove `field :name` from below it:
```elixir
@primary_key {:name, :string, []}  # ADDED THIS LINE
schema "users" do
  #field :name, :string  REMOVED THIS LINE
  field :age, :integer

  timestamps
end
```

**2. migrations/*yyyymmddHHMMSS*_create_user.exs:**
```elixir
def change do
    create table(:users, primary_key: false) do # ADDED "primary_key: false"
      add :name, :string, primary_key: true    # ADDED "primary_key: true"
      add :age, :integer
      timestamps
    end
```

**3. views/user_view.ex:**
```elixir
def render("user.json", %{user: user}) do
    %{# id: user.id,   REMOVED THIS entry
      name: user.name,
      age: user.age
     }
end
```

```console
$mix ecto.migrate
```

Resulting `users` table will have the following schema:
```sql
CREATE TABLE users
(
  name character varying(255) NOT NULL,
  age integer,
  inserted_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,
  CONSTRAINT users_pkey PRIMARY KEY (name)
)
```

Now we have a model with primary key `name` and we can query users by name i.e. `localhost:4000/users/iguberman`

