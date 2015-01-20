mysql-query-composer
====================

An PHP class That allows you to build very complicate queries in a very easy mode

### Initialization and configuration

```php
    $db = new MysqlQueryComposer();
    $db->setConfig('host',(string) @$_config['db']['host']);
    $db->setConfig('user','root');
    $db->setConfig('pass','');
    $db->setConfig('base','your-db-name');
    
    $db->connect();
    $db->connectDb();
```
> #### Use it as a global
> 
> ```php
>   function __db() {
>       global $db;
>       return $db;
>   }
> ```
> so we will use `__db()->...` insead of `global $db;$db->...`

### Simple Queries

#### Select Query

> ```php
>   __db()->select(
>       'tbl_users' => 'users',
>       array(
>           'tbl_users.id'  => 33,
>           'tbl_users.type'    => 'client'
>       ),
>       array(
>           'id'    => 'tbl_users.id',
>           'name'  => 'tbl_users.name',
>           'type'  => 'tbl_users.type'
>       ),
>       "LIMIT 0 , 5");
> ```
> 
> ##### Result Query
> 
> ```mysql
>   SELECT
>       `your-db-name`.`tbl_users`.`id` as `id`,
>       `your-db-name`.`tbl_users`.`name` as `name`,
>       `your-db-name`.`tbl_users`.`type` as `type`
>       FROM
>           `your-db-name`.`users` AS `tbl_users`
>   WHERE
>       `tbl_users`.`id` = "33"
>       AND
>       `tbl_users`.`type` = "client"
>   LIMIT 0 , 5
> ```

#### Update Query

> ```php
>   __db()->update(
>       'users',
>       array(
>           'type'  => "admin",
>           'name'  => "( CONCAT(`name`,'+updated') )"
>       ),
>       array(
>           'and',  // optional row
>           'type'  => "client",
>           'name NOT LIKE "Mark%"',
>           array(
>               'or',
>               'id < 100',
>               'id > 200'
>           )
>       ),
>       "LIMIT 100"
>   );
> ```
> 
> ##### Result Query
> 
> ```mysql
>   UPDATE
>       `your-db-name`.`users`
>   SET
>       `your-db-name`.`users`.`type` = "admin",
>       `your-db-name`.`users`.`name` = ( CONCAT(`name`,'+updated') )
>   WHERE
>       `your-db-name`.`users`.`type`   = "client"
>       AND
>       `your-db-name`.`users`.`name` NOT LIKE 0x4d61726b25
>       AND (
>           `your-db-name`.`users`.`id` < 100
>           OR
>           `your-db-name`.`users`.`id` > 200
>       )
>   LIMIT 100
> ```

#### Delete Query

> ```php
> $_db = new MysqlQueryComposer();
> ...
> $_db->delete(
>         'users',
>         array(
>             'and', 'id > 100', 'type' => 'admin'
>         ),
>         "order by id desc limit 10"
>     )
> ```
>
> ##### Result Query
> ```mysql
> DELETE FROM `main_db`.`users` WHERE id > 100 and type = 'admin' order by id desk limit 10


#### Insert Action

> ```php
>   $db->insert(
>       "users",
>       array(
>           'name'      => 'Foo',
>           'password'  => 'pass'
>       ),
>       " ON kEY DUPLICATE ... " /* query end */
>   );
> ```
> 
> ##### Result Query
> 
> ```mysql
> INSERT INTO `users` ( `name`, `password` ) VALUES ( 'Foo', 'pass' ) ".$query_end
> ```


#### Join Example

> ```php
>   $_db->select(
>       array(
>           'left join',
>           'tbl_users' => 'users',
>           'tbl_assessments' => 'assessments',
>           array(
>               'tbl_assessments.owner_id = tbl_users.id'
>           ),
>           'tbl_access_assessment' => 'assessments--access',
>           array(
>               'tbl_access_assessment.assessment_id = tbl_assessments.id'
>           )
>       ),
>       array(
>           'tbl_access_assessment.user_id' => 29,
>           'tbl_users.type' => 'client'
>       ),
>       array(
>           '*'
>       ),
>       "LIMIT 0 , 5");
> ```
> #### Result Query
> 
> ```mysql
>   SELECT *
>       FROM (
>           `eval-center`.`users` AS `tbl_users`
>       ) LEFT JOIN (
>           `eval-center`.`assessments` AS `tbl_assessments`
>       ) ON ( `tbl_assessments`.`owner_id` = `tbl_users`.`id`
>       ) LEFT JOIN (
>           `eval-center`.`assessments--access` AS `tbl_access_assessment`
>       ) ON ( `tbl_access_assessment`.`assessment_id` = `tbl_assessments`.`id` )
>   WHERE
>       `tbl_access_assessment`.`user_id` = "29"
>       AND `tbl_users`.`type` = "client"
>   LIMIT 0 , 5
> ```

#### Multi select example
> ```php
>       $_db->multiselect(
>           array(
>               array(  // select users
>                   'table' => 'users',
>                   'filter'    => array(
>                           'users.type' => 'client'
>                       ),
>                   'normalize' => 'users.id'
>               ),
>               array(  // select users
>                   'table' => 'users2',
>                   'filter'    => array(
>                           'center != 0'
>                       ),
>                   'normalize' => 'users2.id'
>               )
>           ),
>           "union",
>           "LIMIT 0,10"
>       )->toArray()
> ```
> 
> ##### Result Query
> ```mysql
>       ( SELECT `users`.`id` from `eval-center`.`users` where `users`.`type` = "client"   ) 
>           union
>       ( SELECT `users`.`id` from `eval-center`.`users2` where  `center` != "0"   ) 
>       LIMIT 0,10
> ```