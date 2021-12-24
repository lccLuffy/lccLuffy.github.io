---
layout: post
permalink: /some-tiny-but-useful-laravel-tricks/
title: 一些实用的 Laravel 小技巧
date: 2016-10-29T23:26:00
lang: zh-CN
---

Laravel 中一些常用的小技巧，额，说不定你就用上了。。。<img class="biaoqing" src="https://static.lufficc.com/image/4fb9692a4d7a72204e7fdb966a6f4bf7.png">

#### 1.侧栏

网站一般都有侧栏，用来显示分类，标签，热门文章，热门评论啥的，但是这些侧栏都是相对独立的模块，如果在每一个引入侧栏的视图中都单独导入与视图有关的数据的话，未免太冗余了。。。所以最佳的做法是：新建一个`widgets`视图文件夹，再利用Laravel 的`ViewComposers`单独为侧栏绑定数据，这样侧栏就可以随便引入而不用关心数据是否绑定啦~~~

举个栗子？拿最常用的分类侧栏来说，在`resources/views/widgets`下新建你的分类侧栏视图文件`categories.blade.php`：

```html
<div class="widget widget-default">
    <div class="widget-header"><h6><i class="fa fa-folder fa-fw"></i>分类</h6></div>
    <ul class="widget-body list-group">
        @forelse($categories as $category)
            @if(str_contains(urldecode(request()->getPathInfo()),'category/'.$category->name))
                <li href="{{ route('category.show',$category->name) }}"
                    class="list-group-item active">
                    {{ $category->name }}
                    <span class="badge">{{ $category->posts_count }}</span>
                </li>
            @else
                <a href="{{ route('category.show',$category->name) }}"
                   class="list-group-item">
                    {{ $category->name }}
                    <span class="badge">{{ $category->posts_count }}</span>
                </a>
            @endif
        @empty
            <p class="meta-item center-block">No categories.</p>
        @endforelse
    </ul>
</div>
```

新建`app/Http/ViewComposers`文件夹，然后创建`CategoriesComposer.php`：

```php
<?php
namespace App\Http\ViewComposers;
use App\Http\Repositories\CategoryRepository;
use Illuminate\View\View;
class CategoriesComposer
{
    public function __construct(CategoryRepository $categoryRepository)
    {
        $this->categoryRepository = $categoryRepository;
    }

    public function compose(View $view)
    {
        $categories = $this->categoryRepository->getAll()->reject(function ($category) {
            return $category->posts_count == 0;
        });
        $view->with('categories', $categories);
    }
}
```

再在`app/Providers`文件夹下新建`ComposerServiceProvider.php`文件：
```php
<?php
namespace App\Providers;
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\View;
class ComposerServiceProvider extends ServiceProvider
{

    public function boot()
    {
        View::composer('widget.categories', 'App\Http\ViewComposers\CategoriesComposer');
    }

    public function register(){}
}
```
最后别忘了在`config/app.php`中的`providers`数组中添加`App\Providers\ComposerServiceProvider::class`啊。好了，现在你可以随时随地`@include('widget.categories')`了。对了，要善于在`ViewComposer`中利用`Collection`的[强大方法](https://laravel.com/docs/5.3/eloquent-collections#available-methods)进行数据处理幺~~

#### 2.善用路由别名

Laravel 最让人喜欢的地方之一是可以给路由起一个别名，比如：
```php
Route::get('user/profile', 'UserController@showProfile')->name('user.profile');
// 等价于：
Route::get('user/profile', ['uses' => 'UserController@showProfile' , 'as' => 'user.profile']);;
```
然后，就可以在试图中就可以使用`route()`方法引用了：
```php
// 例如：
<a href="{{ route('user.profile') }}">lufficc</a>
```
因为一个普通的项目路由至少也得有几十个，如果使用`url()`方法的话，你不但要记住具体的路由，更麻烦的是如果你将来想要改变某个路由（比如把`'user/profile'`改为`'u/profile'`，或者加个前缀啥的），必须改变所有相关的视图文件，这。。。这。。。不敢相信，而使用命名路由的话，只要命名不变，毫不受影响。

所以视图文件中尽量避免使用`url()`方法，为每一个路由命名，一个默认的命名规则为：`资源名称.<属性>或者<动作>`，如`post.show`，`image.upload`。


#### 3.全局动态设置

仅仅是`.env`的配置还无法满足我们的需求，有时我们需要可以在后台动态的进行一些设置，比如网站的标题，网站的背景图片或者是否允许评论等等。那么实现这个的最佳实践是什么？

熟悉wordpress的同学知道，wordpress可以进行很多自定义，因为wordpress有一张键值对数据库表，它就是靠这个实现个性化的。因此我们也可以参考这种思路，增加一个键值对表，以[Xblog](https://github.com/lufficc/Xblog)为例子，新建一个`maps`表：

```php
Schema::create('maps', function (Blueprint $table) {
       $table->increments('id');
       $table->string('key')->unique();
       $table->string('tag')->index();
       $table->text('value')->nullable(true);
});
```
`maps`表的作用就是实现键值对`key-value`存储，`tag`的是为了可以有一个分类。然后后台进行存储的话，不要写死，这样就可以随时在变单中添加设置而无需更改代码：

```php
$inputs = $request->except('_token');
foreach ($inputs as $key => $value) {
            $map = Map::firstOrNew([
                'key' => $key,
            ]);
            $map->tag = 'settings';
            $map->value = $value;
            $map->save();
}
```

注意`firstOrNew`的用法：如果不存在这个选项我们就新增一个并保存，否则就更新它。然后我们就可以在视图中随便增加任意多个表单了（或者也可以用js动态生成表单）。有了数据，怎么在视图中利用呢？利用`ViewComposer`，新建一个`SettingsComposer.php`，然后将查询的数据以数组的形式传递给试图：

```php
//在SettingsComposer.php的compose方法中绑定数据
public function compose(View $view)
{
    $settings = Map::where('tag', 'settings')->get();
	$arr = [];
    foreach ($settings as $setting) {
      $arr[$setting->key] = $setting->value;
    }
   $view->with($arr);
}
```

然后就可以在视图中随便引用了，如你表单新增加了一个`description`

```php
<input type="text" name="description" value="{{ $description or ''}}">
```
然后就可以在任何视图引用了:`{{ $description or ''}}`。另外还可以绑定一个单例`Facades`到容器，这样就可以在代码中随时获取配置信息啦~~~
比如：
```php
//1.注册
public function register()
{
    $this->app->singleton('XblogConfig', function ($app) {
       return new MapRepository();
   });
}
//2.注册Facade
class XblogConfig extends Facade
{
    public static function getFacadeAccessor()
    {
        return 'XblogConfig';
    }
}
//3.添加到aliases数组


'aliases' => [

        *****************  省略  *************************
        'XblogConfig' => App\Facades\XblogConfig::class,
    ],
		

//4.愉快的使用，可爽
$page_size = XblogConfig::getValue('page_size', 7);

```

#### 4.数据库查询

怎么统计一篇文章有多少评论？最快的方法是：
```php
$post = Post::where('id',1)->withCount('comments')->first();
```
这样`$post`变量就有一个属性`comments_count`了：
```php
$post->comments_count;
```
如果想获取点赞数大于的100的评论个数怎么办？这样：
```php
$post = Post::where('id',1)->withCount('comments',function($query){
       $query->where('like', '>', 100);
   })->first();
```
简单吧~~

#### 5.多态关联

文章可以有评论，页面可以有评论，评论也可以有评论，但是总不能建三张评论表吧？如果自己写条件判断也太麻烦了吧。。。Laravel的多态关联上场了！！
```php
//1.第一步在Comment模型中说明我是可以多态的
public function commentable()
{
    return $this->morphTo();
}

//2.在想要评论的模型中增加comments方法，
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}

//3.使用，就像普通的一对多关系一样：
$model->comments;
```
原理很简单，`comments`表中增加两个列就行：
```php
Schema::create('comments', function (Blueprint $table) {
     ***************省略*******************
     $table->morphs('commentable');
     //等价于
     $table->integer('commentable_id')->index();
     $table->string('commentable_type')->index();
    ****************省略******************
});
```
然后 laravel 会自动维持这些关系。注意，保存的评论的时候是有小技巧的，你的表单中至少要传两个参数：`commentable_id`和`commentable_type`：
```php
$comment = new Comment();

$commentable_id = $request->get('commentable_id');
//commentable_type取值例如：App\Post，App\Page等等
$commentable = app($request->get('commentable_type'))->where('id', $commentable_id)->firstOrFail();

****************省略******************

$commentable->comments()->save($comment);
```

保存评论的时候并不知道是谁的评论，而是使用容器根据`commentable_type`生成一个模型实例，这样也就和具体的模型解耦了，你可以让任何东西可以评论，而不需要修改代码。


#### 6.缓存优化相关

如果你想要在`.env`文件中添加自己的配置，记住一定要在`config`文件夹下某个配置文件的数组中添加对应的。记住，除了`config`文件夹下的配置文件，永远不要在其它地方使用`env`函数，因为部署到线上时，配置文件缓存（`php artisan config:cache`）后，`env`函数无法获得正确的值。

另外注意的是，路由文件中尽量不使用闭包函数，统一使用控制器，因为缓存路由的时候`php artisan route:cache`，无法缓存闭包函数。

#### 7.Redis

如果你缓存使用Redis，`session`也使用了Redis，队列已使用了Redis，这样没问题，速度很快，但是！！当你运行`php artisan cache:clear`清除缓存时，会把你的登录信息清除，也会把队列清除。。。这就不优雅了。解决办法很简单，为它们分配不同的连接即可。
首先在`config\database.php`中增加连接，注意`database`序号：
```php
'redis' => [

        'cluster' => false,

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],
        'session' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 1,
        ],
        'queue' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 2,
        ],

    ],
```

然后分别为`session`和`queue`更换连接：
```php
//queue.php中的connections数组中：
'redis' => [
            'driver' => 'redis',
            'connection' => 'queue',
            'queue' => 'default',
            'retry_after' => 90,
        ],
				
//session.php中的connection选项：
'connection' => 'session',
```
这样他们就互不相干了~~


***以上经验来自 [Xblog](https://github.com/lufficc/Xblog)，示例均可以在 [Xblog](https://github.com/lufficc/Xblog) 找到***
