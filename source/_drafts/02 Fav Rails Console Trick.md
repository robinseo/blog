---
title: Favorite Rails Console Trick
categories:
- Rails
---

### 콘솔에 여러줄 코드 복사/붙여넣기 망하지 않는 법

가끔 많은 줄의 코드를 Rails Console에 복사/붙여넣기하면 제대로 읽지 못하거나, 혹은 너무 버벅거려서 터미널을 꺼버리셨던 경험이 있으셨나요?

```shell
$ rails c -- --nomultiline
```

### 콘솔에서 결과값을 자동으로 출력하는 것 그만 좀 하게 하기

rails console도 결국 REPL 위에서 돌아가는 거라 evaluation 한 것을 출력하는 것을 기본으로 하고 있어요.

가끔 아래 코드처럼 콘솔에서 대량의 자료를 Pretty Print로 보고싶은데, 그 뒤로 지저분한 결과값이 따라 올라올 때가 있는데요.

```ruby
pp User.where("User Complex query opts")
```

이런 트릭으로 evaluation 값을 nil로 만들어서 출력을 nil이 찍히게 해줄 수 있습니다.

```ruby
pp User.where("User Complex query opts"); nil
```

정말 두 번 다시 이 Evaluation echo가 보기 싫다면 이렇게 콘솔을 켜도 됩니다.

```shell
$ rails c -- --noecho
```

nomultiline이나 noecho나 둘 다 irb의 config라서 global irb config를 만져볼 수도 있고, 콘솔을 켤 때 마다 argument를 넘겨줄 수 도 있습니다. 혹은 Rails
console에서 IRB.conf 객체를 접근해서 바로 조정할 수도 있어요.

### Sandbox Mode & Transaction

Rails Console에는 샌드박스 모드가 있습니다.

샌드박스 모드에서 실행한 모든 ActiveRecord 쿼리는 전부 무효로 돌아갑니다.

```shell
$ rails console --sandbox
$ rails c -s
```

피치 못할 사정으로 운영 DB를 급하게, 대량 조작해야 하는데 자신이 없다? 근데 빨간불의 상황이다? 그런데 Alpha 환경에서는 도저히 검증 해볼 환경이 안된다? 이 경우에 딱
샌드박스 모드로 Dry Run을 해볼 수 있습니다. RDB로 나가는 쿼리만 안나가게 해주는거지 Model Callback 같은건 그대로 나가게 되니 skip_callback 같은게 필요한지 잘 보면 좋고요.

또 하나는 샌드박스 모드를 응용해서 Plain Rails Code 를 가지고 테스트를 돌려볼 수 있는 간단한 도구를 만들어 볼 수 도 있어요.

RSpec이나 MiniTest 만큼 강력하진 않지만, 작성하고 있는 Rails 코드를 그대로 갖다 붙여서 테스트를 해볼 수 있어서 좋습니다.

Rails Sandbox 가 어떻게 돌아가는지 궁금하실텐데, 정확히는 저도 잘 모르고요.. DB Abstract Adapter 가 구현해둔 TransactionManager에 의해 알아서 SavePoint를 잡아서
스택에 차곡 차곡 쌓아두고 있는 건 구경해 볼 수 있어요.

```ruby
User.create!
User.create!

#위에서 두 번 Transaction을 일으켜서, stack 안에 Transaction이 두 개 쌓여있어요.
transaction = User.connection.instance_variable_get(:@transaction_manager).instance_variable_get(:@stack)
# => The Array of ActiveRecord::ConnectionAdapters::RealTransaction, size = 1
prev_transaction = User.connection.instance_variable_get(:@transaction_manager).instance_variable_get(:@stack)
```





