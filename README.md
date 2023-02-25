# ed-improve-laravel-migrations
Tips for Laravel migrations

## Don't hardcode table name in migration

When you’re defining a model migration, probably you write something like this:

```
Schema::create('TABLE_NAME', function (Blueprint $table) {
    ...
}
```

I don’t enjoy this, because the table name is not linked to the model. Instead, I define the property ```$table``` into the model:

```
class MyModel extends Model {
    protected $table = 'TABLE_NAME';
    ...
}
```

And I get the table name dynamically in the migration using the (non-static) method ```getTable()```:

```
Schema::create((new MyModel())->getTable(), function (Blueprint $table) {
    ...
}
```

In this way, you will know every time the name of your table without seeing the database, and you create a link between the model and the database.

## Use enums

Enums are a data type that allows you to specify a set of predefined values that a column can hold. Essentially, **an enum is a list of named values that represent the allowed values for a column in a table**. For example, you could define an enum for a column named “status” with the values “pending” and “active”. When you insert data into this column, the value must be either “pending” or “active”. If you attempt to insert any other value, MySQL will throw an error.

You can define an enum also inside migrations:

```
Schema::create((new MyModel())->getTable(), function (Blueprint $table) {
    $table->enum('status', ['pending', 'active']); // status can be only pending or active
}
```

## Tip: Enum casting

Starting from PHP 8.1, you can define an enum in PHP. In Laravel, you can say “this model attribute is an enum” so your application will cast the attribute to a specified enum value. Let’s see how.

Define your enum, for example, the UserStatus, and assume that it can be only a string.

```
namespace App\Enums;
enum UserStatus: string {
  case Pending = 'pending'; // Pending is the name, 'pending' is the value
  case Active = 'active';
}
```

In your User model, you can cast the status to this enum through the $casts array:

```
use App\Enums\UserStatus;
class User extends Model {
  protected $casts = [
    'status' => UserStatus::class,
  ];
}
```

Now, every time you will access the property status, it will return an instance of UserStatus.

```
// assume status=pending into the database
return $user->status === UserStatus::Pending; //true
```

## Manage belongsTo relationships without hardcoding column names

Imagine you have a belongsTo relation, for example between the user and his customers. A user can have many customers, but a customer belongs only to one user. In this case, typically you will write:

```
Schema::create((new Customer())->getTable(), function (Blueprint $table) {
    $table->unsignedBigInteger('user_id'); 
    $table->foreign('user_id')->references('id')->on('users');
}
```

You can do better. You don’t have a link between models, and it could lead to many problems. For example, imagine you have two kinds of users (e.g. UserAdmin and UserModerator) and you have a ```user_id``` in your column. What will this column reference?

You can pass a model instead of hardcoding everything. There is a method called ```foreignIdFor``` where you can pass a model class (e.g. the User class) that will create the column ```user_id``` automatically, and you can chain the method constrained to add the foreign constraint.

So, you can write the previous statement like this:

```
Schema::create((new Customer())->getTable(), function (Blueprint $table) {
    $table->foreignIdFor(User::class) //add the column to customer table
          ->constrained(); //add the foreign constraint
}
```
