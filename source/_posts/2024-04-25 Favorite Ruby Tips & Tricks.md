---
title: Favorite Ruby Tips & Tricks
date: 2024-04-25
categories:
- [rails]
tags:
- [Ruby]
- [Rails]
---

### Conditional Method Chaining

Method Chain을 조건에 따라 조정하는 경우가 있습니다.

예를 들자면 아래와 같이 조건에 따라 유저 목록을 필터링해서 가져오는 경우를 들어 볼 수 있겠어요.

```ruby
name, email = params.values_at(:name, :email)

user = User.all
user = user.name_like(name) if name
user = user.email_like(email) if email
user = user.page(page_params[:page]).per(page_params[:per])
```

then Enumerator를 이용해서 조금 더 FP 스러운 느낌을 내 볼 수 있어요.
이쪽이 조금 더 가독성이 좋다고 느끼는 편입니다.

```ruby
name, email = params.values_at(:name, :email)

users = User.all
            .then { |users| name ? users.name_like(name) : users }
            .then { |users| email ? users.email_like(email) : users }
            .page(page_params[:page])
            .per(page_params[:per])
```

then은 최상위 Object에 구현되어있는 method라서 NilClass도 가지고 있습니다. 
nil guard 없이도 편하게 사용할 수 있는 장점이 있어요.

### Optional Element in Array & Hash
Flag에 따라 Array나 Hash에 Element를 넣거나 없애고 싶을 때 one-liner로 정리 할 수 있어요.

```ruby
def flag? = false

array = [
  "1st element",
  *("2nd element" if flag?),
  "3rd element"
]

# => ["1st element", "3rd element"]

hash = {
  a: 1,
  **(flag? ? { b: 2 } : {}),
  c: 3,
}

# => {:a=>1, :c=>3}
```

### Hash with default value

가끔 Tree 같은걸 만들거나 할 때 요긴하게 쓰이는 default value를 가지는 Hash를 one-liner로 만들 수 있어요.

```ruby
# Default Value = {}
Hash.new { |h, k| h[k] = Hash.new &h.default_proc } # Infinite nested hash

# Default Value is Your value
Hash.new { |h, _| h = { "foo" => 0, "bar" => 0 } }
```

Ruby Hash는 다른 언어와는 약간 다르게 주어진 Key로 Value를 찾을 수 없을 때 기본적으로 nil을 반환하게 되어 있는데요. 이 동작을 수정하려면 Hash의 default_value를 설정해주거나, 더 fine하게 바꾸려면 default_proc을 설정해줄 수 있어요.

위의 예시들은 모두 default_proc을 설정해 준 케이스예요.

### Inline rescue

간단하고 별거아닌 에러 핸들링은 inline rescue가 좀 쓸만합니다.

3rd gem 중에 쓸데없이 에러를 터뜨리는 것들이 있어요. 대표적으로 Python 코드를 그대로 옮겨놓은 AWS Gem이 그런데요. Ruby에서는 Array에 IndexError가 없는데, AWS Gem은 Index를 터뜨린다거나 약간 몇군데가 Python 처럼 돌아갑니다. 

```ruby
foo = Foo.complex_fetching(...) rescue nil
```


### 소소하지만 좋은 Enumerator

제가 즐겨 쓰는 Enumerator method 몇가지를 남겨둘게요

* times (요거 자주 씁니다 3.times { puts "Hello!" })
* inject (숫자 더할때 많이..)
* each_with_object (요게 js의 reduce와 좀 비슷한 느낌이라 가끔씩)
* with_index (인덱스가 안달리는 경우가 있을때 체인 걸어서 쓸 때가 가끔씩)
* each_with_index
* tap (Enumeration 돌때마다 뭔가를 터치할때 요긴한데, 생각보다 쓸데가 없는.. 하지만 유용한 순간엔 유용한..)


### tally

Ruby 2.7 에 새로 생긴 소소한 method 중 tally 가 있습니다. 간단한 group count method 인데요. 간단한 CS 문의를 봐줄 때나, 통계 데이터 뽑을 때 요긴하게 쓰고 있습니다.


```ruby

bad_user_ids = [10, 11, 11, 12, 15, 15, 13, 14, 1, 2, 11, 1]
bad_user_ids.tally
# => {10=>1, 11=>3, 12=>1, 15=>2, 13=>1, 14=>1, 1=>2, 2=>1}
```


### Method Digging

런타임 시점에 만들어진 Method의 이름과 위치를 알아내는 테크닉에 대해 소개합니다.

#### 1

가끔 어떤 클래스에 어떤 메서드가 실제로 존재하는지 직접 확인해야 하는 경우들이 생기곤 합니다.

왜냐면 소스코드에는 없지만, 런타임에 갑자기 메서드가 생기는 경우들이 있으니까요. 

ps. 메타프로그래밍은 훌륭한 도구이지만 다음 사람을 위해 메서드 시그니쳐를 타이핑해주거나 주석 정도는 남기도록 해요....

```ruby
User.methods # 이 클래스의 클래스 메서드 이름들을 Array of symbol로 리턴합니다.
User.instance_methods # 이 클래스의 인스턴스 메서드 이름들을 Array of symbol로 리턴합니다.
```

그런데, User 라는 ActiveRecord Class는 엄청 많은 클래스와 모듈을 상속 / 믹스인 받아온 상태라 우리가 알고 싶지 않은 메서드 이름을 까지 리턴해주게 돼요.
심플하게

```ruby
User.methods - Object.methods
```
또는

```ruby
User.methods - ApplicationRecord.methods 
```
이정도로 Ancestor methods들을 제거해서 볼 수 있습니다. 혹은 User.ancestors 를 통해 이 클래스의 Ancestor chain 을 확인해 본 다음에 원하는 위치부터 method를 제거해 볼 수 도 있겠어요.


#### 2
메서드의 이름을 찾아냈다면, 이제 그 메서드가 어디에 있는지, 어떤 코드를 가지고 있는지도 찾아 볼 수 도 있습니다.

```ruby

User.method(:find)
# => #<Method: #<Class:User(...)>(ActiveRecord::Core::ClassMethods)#find(*ids) /.../activerecord-6.1.7.7/lib/active_record/core.rb:337>

User.method(:find).source
# 진짜 find method의 소스 코드를 출력해줍니다.

User.method(:find).source_location
# ["/.../activerecord-6.1.7.7/lib/active_record/core.rb", 337] 몇번째 파일 어디 라인인지 알려줍니다.
```


### Ruby Binding

사실 실무를 하면서 쓸일이 있나 싶기는 하지만..

Rails 온보딩을 해드리면 종종 받는 질문 중 하나라서 남겨봅니다.

`app/controllers/users_controller.rb`
```ruby
class UsersController < ApplicationController
  def show
    @user = "Robin"
  end
end
```

`app/views/users/show.html.erb`
```irb
Hi <%= @user %>!
```

어떻게 erb는 @user를 알아서 Robin으로 바꿔치기 해가는 걸까요?

옛날 옛적 Spring의 경우 ModelAndView 객체에 명시적으로 담아가게 되어있는데, Rails엔 그런게 다 생략 되어 있죠.  erb의 instance variable 바꿔치기 를 설명하기 전에 binding에 대한 이야기를 잠깐 소개 하고 갈게요.

Ruby는 Javascript와 비슷하게 Context Binding 개념이 있는데요. 그레서 Lexical Scope라는 표현을 종종 사용하곤 합니다.  이 얘기는 너무 기니깐.. 아래 처럼 Lexical Scope을 맘대로 바꿀 수 있습니다. 진짜 Javascript 느낌이 나죠..?

```ruby
name = "robin"
eval("puts name", binding) # robin


def get_binding
  name = "llama"
  binding
end

eval("puts name", get_binding) #llama
```

다시 돌아와서, erb는 아래와 같은 방식으로 interpolation을 합니다. (많이 축약해놨어요.)

```ruby
class UsersController < ApplicationController
  def show
    @user = "Robin"
    
    erb = ERB.new("Hi <%= @user %>!") # 실제로는 convention에 의해 Controller#Action과 같은 위치의 ERB 파일을 읽어옵니다.
    render html: erb.result(binding) #그리고 이 컨트롤러의 lexical scope을 그대로 erb 객체에게 넘겨줍니다. 
  end
end
```

### Instance debugging

Ruby는 클래스의 모든 부분에 있어 열려있습니다. 런타임에 변수나 메서드를 지우고, 새로 만들고, 덮어씌울 수 있습니다.  루비를 꽤 오래 써 온 저한테는 조금 익숙한 개념인데, 지금 잠깐 생각해보니 "그래도 되는거야..?" 싶은 개념 인 것 같기도 하네요. 

여튼 이러한 맥락이 있어서 그런지, Ruby에서는 실제로는 접근제어자를 통해 접근하지 못하는 변수나 메서드도 실제로는 접근 할 수 있습니다. 이런 부분들이 가끔 디버깅 할 때 요긴하게 쓰여서 소개해드리려 합니다.


#### [1] private method call

private modifier로 보호받고 있는 method를 곧 죽어도 실행하고 싶은 경우가 가끔 있습니다.

그럴 땐 `send`를 이용해 호출 할 수 있습니다.  `foo.bar` 대신 `foo.send(:bar)` 요렇게요.


#### [2] instance_variable_get

보통은 attr_reader 또는 attr_accessor 같은 매크로를 통해 getter를 만들어줘야 해당 클래스 내부의 instance variable 을 외부에서 사용 할 수 있는데요.

사실은 강제로도 꺼내서 쓸 수 있습니다. 

```ruby
class Foo
  def initialize(bar) = @bar = bar
end

class FooBar
  attr_reader :bar
  def initialize(bar) = @bar = bar
end



foo = Foo.new(1)
foobar = FooBar.new(1)

foo.bar # It raises NoMethodError
foobar.bar # 1
foo.instance_variable_get(:@bar) # 1
```

사실 RubyMine 디버거가 있어서 쓸 일이 자주는 없지만, 특정 구현체 하나만 가지고 콘솔에서 디버깅을 할 때에는 이게 좀 유용합니다.
