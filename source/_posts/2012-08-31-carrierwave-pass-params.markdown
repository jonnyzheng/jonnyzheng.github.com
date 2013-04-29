---
layout: post
title: "carrierwave uploader 中传递变量"
date: 2012-08-31 16:29
comments: true
categories: rails
---


在carrierwave uploader中经常会用到version的功能，一般我们的写法都是在里面写死process的一些参数，比如

```ruby
  version :small do
    process :resize_to_fill => [48,48]
  end
```

但是有的时候可能我们需要动态的传递一些参数进来对图片做相应的操作，比如用户在前端选择图片的头像区域，然后传到后台进行切割，这个时候需要加一个process方法, 下面的crop_area方法通过参数来截取图片中对应的区域。

```ruby
 def crop_area
    manipulate! do |img|
      ...
    end
  end
```



但是参数如何传进来成了问题，起初我是在 Uploader里定义了一个可访问的变量,然后在controller里将参数传递给Uploader实例，结果在crop_area里根本就访问不到Uploader里的实例变量，后来看了下源码发现carrierwave的实现逻辑貌似不能用这样的方法。于是用了一个曲线救国的方法，通过Model来传递参数：


```ruby
# model class
# define attr_accessor coords
class User < ActiveRecord::Base
  attr_accessor :coords
  mount_uploader :icon, AvatarUploader
end



# controller
# pass the params to @user.coords
def crop_icon
  @user.coords = params[:coords]
  @user.icon = File.open(path)
  @user.save
  redirect_to :action=> 'basic'
end



# Uploader
# the model in the function is same as @user in controll,
# and can be invoked inside of process method 
 def crop_area
    manipulate! do |img|
      unless model.coords.nil?
        coords = JSON.parse(model.coords)
        img.crop("#{coords['w']}x#{coords['h']}+#{coords['x']}+#{coords['y']}")
      end
      img = yield(img) if block_given?
      img
    end
  end
```


所以最后通过在 Model 中设置实例变量解决动态传递参数的问题。