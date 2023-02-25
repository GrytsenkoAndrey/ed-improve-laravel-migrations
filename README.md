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
