---
title: laravel 5.1 Model 属性详解
date: 2017-12-30 10:00:00
tags: [Laravel, PHP]
---

```php
<?php
 
namespace Illuminate\Database\Eloquent;
 
/**
 * 下面提到某些词的含义：
 *      1、覆盖： 在继承该类 \Illuminate\Database\Eloquent\Model 的自定义的模型类中， 定义一个同名 field，值不一样
 */
abstract class Model1 implements ArrayAccess, Arrayable, Jsonable, JsonSerializable, QueueableEntity, UrlRoutable
{
    /**
     * 数据库连接，我们在 config/database.php 里面的 connections 数组定义的连接。
     * usage：
     *      (new xxModel)->connection('mysql')
     *      DB::connection('pgsql')
     *      或者在模型定义里面覆盖该属性，如： protected $connection = 'pgsql';
     */
    protected $connection;
 
    /**
     * 模型关联的数据库表名，默认是 模型名->下划线命名->复数形式，
     * 好比如定义了 User 模型，class User extend Model，那么默认的表名是 users。
     *
     * usage：
     *      (new xxModel)->setTable('xxx')
     *      (new xxModel)->getModel()
     *      或者在模型定义的时候覆盖该属性，如： protected $table = 'tb_user';
     */
    protected $table;
 
    /**
     * 主键字段，默认为 id，也可以覆盖该属性
     */
    protected $primaryKey = 'id';
 
    /**
     * 不知道哪里用到，除了该文件的 getter 和 setter
     */
    protected $perPage = 15;
 
    /**
     * 主键是否默认自增长
     */
    public $incrementing = true;
 
    /**
     * 是否由模型去维护时间戳字段，如果我们想手动去维护，可以设置为 false
     * usage：
     *      默认的时间戳字段是 created_at、updated_at，
     *      我们如果想要使用自定义的字段，则要在模型里面覆盖 CREATED_AT、UPDATED_AT 这两个常量(下面有定义)
     * 其他：
     *      默认使用 mysql 的 datetime 类型，如果需要更改为 10 位整型，可以设置 protected $dateFormat = 'U'; ()
     */
    public $timestamps = true;
 
    /**
     * 我们在给模型对象设置非 public 属性的时候，会通过 setAttributes 方法，保存到该数组中
     * usage：
     *      $user = new User();
     *      $user->name = 'laravel';
     *      User 中没有定义 public $name 的时候， $attributes 就会多了 'name' => 'laravel' 的键值对
     */
    protected $attributes = [];
 
    /**
     * 保存模型的原始数据，后续修改模型属性只会修改 $attributes，以便侦测变化
     */
    protected $original = [];
 
    /**
     * 模型的关联数据
     */
    protected $relations = [];
 
    /**
     * 隐藏的属性，我们调用模型的 toArray 方法的时候不会得到该数组中的属性，
     * 如果需要也得到隐藏属性，可以通过 withHidden 方法
     */
    protected $hidden = [];
 
    /**
     * 与 hidden 数组作用差不多，共同作用
     */
    protected $visible = [];
 
    /**
     * 其中一个用法，根据现有某几个属性，计算出新属性，并在 模型 toArray 的时候显示
     * usage：
     *      模型里面定义： protected $appends = ['full_name'];
     *      public function getFullNameAttribute() { return $this->firstName . ' ' . $this->lastName; }
     */
    protected $appends = [];
 
    /**
     * mass assignment 的时候可以批量设置的属性，目的是防止用户提交我们不想更新的字段
     * 注意：
     *      和 $guarded 同时使用的时候， $guard 设置的会无效
     */
    protected $fillable = [];
 
    /**
     * 不能批量赋值的属性
     */
    protected $guarded = ['*'];
 
    /**
     * 需要进行时间格式转换的字段
     * 应用场景：
     *      一般情况我们只定义了 created_at、updated_at，我们还可能会保存用户注册时间这些，register_time，
     *      这样我们就可以定义，protected $dates = ['register_time'];
     * 好比如：
     *      我们定义的 $dateFormat 为 mysql 的 datetime 格式，我们即使把 register_time 设置为 time(),
     *      实际保存的其实是 datetime 格式的
     */
    protected $dates = [];
 
    /**
     * created_at、updated_at、$dates数组 进行时间格式转换的时候使用的格式
     */
    protected $dateFormat;
 
    /**
     * 自动格式转换，定义方式： protected $casts = ['info' => 'json'];
     * 所有可用格式： int、integer、real、float、double、string、bool、boolean、
     *              object、array、json、collection、date、datetime
     * 应用场景：
     *      我们想要保存某些特殊格式到数据库(如json、object)，我们可以使用该数组对我们的数据进行自动转换,
     * usage:
     *      $user = User::create([
     *          'name' => 'ruby',
     *          'info' => ['city' => 'Guangzhou']
     *      ]);
     *
     *      $query_user = User::find($user['id']);
     *      dd($query_user->info);
     * 这里我们可以看数据库保存的 info 字段，实际上是 json 编码的，并且取出来的是 json 解码后的数据
     */
    protected $casts = [];
 
    /**
     * 需要同步更新 updated_at 的关联，调用 save 方法的时候会更新该数组里面定义的关联的 updated_at 字段
     */
    protected $touches = [];
 
    /**
     * todo 自定义事件，目前还没用过  -_-
     */
    protected $observables = [];
 
    /**
     * 需要预加载的关联
     */
    protected $with = [];
 
    /**
     * The class name to be used in polymorphic relations.
     *
     * todo 也不知道
     */
    protected $morphClass;
 
    /**
     * 模型是否存在
     */
    public $exists = false;
 
    /**
     * 判断模型是否是当前请求插入的
     */
    public $wasRecentlyCreated = false;
 
    /**
     * 模型属性名是否是下划线形式的
     */
    public static $snakeAttributes = true;
 
    /**
     * 用来建立数据库连接
     *
     * @var \Illuminate\Database\ConnectionResolverInterface
     */
    protected static $resolver;
 
    /**
     * 用来分发模型事件
     *
     * @var \Illuminate\Contracts\Events\Dispatcher
     */
    protected static $dispatcher;
 
    /**
     * 模型 new 的时候调用模型的 bootXXX 方法， XXX 是模型名称
     */
    protected static $booted = [];
 
    /**
     * 全局查询条件，需要定义
     *
     * @see \Illuminate\Database\Eloquent\ScopeInterface
     */
    protected static $globalScopes = [];
 
    /**
     * 批量设置属性的时候是否不需要筛选字段
     */
    protected static $unguarded = false;
 
    /**
     * 缓存 getXXAttribute 的值
     */
    protected static $mutatorCache = [];
 
    /**
     * 多对多关联方法，不知道哪里用了
     */
    public static $manyMethods = ['belongsToMany', 'morphToMany', 'morphedByMany'];
 
    /**
     * 创建时间戳字段名称
     */
    const CREATED_AT = 'created_at';
 
    /**
     * 更新时间戳字段名称
     */
    const UPDATED_AT = 'updated_at';
}
```
