### Laravel Transformer 重复查询的问题

之前一直觉得Laravel慢，确实比ThinkPHP之类的慢，用了Transformer以后，发现更慢。但是之前一直做项目，也没有去深究，隐隐约约中感觉，如果是一个列表，Transformer中的查询会重复查询。今天打印了一下数据库，确实是。但这功能用的人这么多，理论上不应该出现这样重大的性能消耗啊，百度了一波，发现确实提这个问题的人也很少。so，研究了一下，用到关系查询的时候，应该在查询时直接用With



同一个`Transformer`

```php
//Transformer
public function transform(Administrator $model)
    {
        return [
            'id'         => (int) $model->id,
			'username'=>$model->username,
			'name'=>$model->name,
            /* place your other model properties here */
			'roles'=>$model->permissions,
            'created_at' => $model->created_at->toDateString(),
            'updated_at' => $model->updated_at->toDateString()
        ];
    }
```



使用`with`后只会产生两条查询

```php
//控制器
$user = Administrator::with(['permissions'])->get();
return response()->json(Translate::collection($user,new UserTransformer()));
```



```sql
select * from `admin_users`
select `admin_permissions`.*, `admin_user_permissions`.`user_id` as `pivot_user_id`, `admin_user_permissions`.`permission_id` as `pivot_permission_id` from `admin_permissions` inner join `admin_user_permissions` on `admin_permissions`.`id` = `admin_user_permissions`.`permission_id` where `admin_user_permissions`.`user_id` in (1, 2, 3)
```



不使用`with`会产生N条查询

```php
//控制器
$user = Administrator::get();
return response()->json(Translate::collection($user,new UserTransformer()));
```



```sql
select * from `admin_users`
select `admin_permissions`.*, `admin_user_permissions`.`user_id` as `pivot_user_id`, `admin_user_permissions`.`permission_id` as `pivot_permission_id` from `admin_permissions` inner join `admin_user_permissions` on `admin_permissions`.`id` = `admin_user_permissions`.`permission_id` where `admin_user_permissions`.`user_id` = 1
select `admin_permissions`.*, `admin_user_permissions`.`user_id` as `pivot_user_id`, `admin_user_permissions`.`permission_id` as `pivot_permission_id` from `admin_permissions` inner join `admin_user_permissions` on `admin_permissions`.`id` = `admin_user_permissions`.`permission_id` where `admin_user_permissions`.`user_id` = 2 
select `admin_permissions`.*, `admin_user_permissions`.`user_id` as `pivot_user_id`, `admin_user_permissions`.`permission_id` as `pivot_permission_id` from `admin_permissions` inner join `admin_user_permissions` on `admin_permissions`.`id` = `admin_user_permissions`.`permission_id` where `admin_user_permissions`.`user_id` = 3 
```

----

