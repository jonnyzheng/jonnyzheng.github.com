--- 
categories: []
comments: true
layout: post
published: false
status: draft
tags: []
title: "Rails Testing"
type: post
---
Rails 框架从一开始就很好的支持各种测试(单元测试，集成测试),  到现在又有很多优秀的plugin提供各方面的补充，在我们的项目里用到了RSpec + FactoryGirl(是不是很性感) + autotest, 说一说使用经验供大家参考:

开始之前说点跑题的话，Rails 当时是深受敏捷开发的影响，所以测试功能是一点都不含糊的，但是就TDD来说我还是非常不喜欢的。首先TDD要求先写测试用例，在初期开发中会有很多变数，很多你写的测试用例可能在以后需要修改，而产品没有成型之前做这样的工作有点得不偿失，其次是在没有实际东西之前的凭空想象有点折磨人，我的脑细胞还有别的用处，不想浪费在这方面。再就是个人认为测试代码是一个持续长久的过程，我们没必要通过测试来驱动我们的代码工作，但是我们需要通过持续的提高测试覆盖率来保证我们的代码质量，提高我们的测试效率，可以让我们有更多的时间做更重要的事情，而不是陷入无数bug的泥沼里。
<h3>Rspec</h3>
 

 

 

 
<h3>FactoryGirl</h3>
有了这个工厂，可以很方便的构造不同的模拟数据来运行测试代码。还是上面的例子，如果要测试
{% codeblock %}
absence_at
{% endcodeblock %}
方法，涉及到多个model：
<ul>
<li>HistoryRecord：User的上课记录</li>
	<li>Calendar：User的课程表</li>
	<li>Logging：User的日志信息</li>
</ul>
如果不用factory-girl构造测试数据，我们将不得不在fixture构造这些测试数据。在fixture构造的数据无法指定是那个测试用例使用，但是如果用Factory的话，可以为这个方法专门指定一组测试数据。

{% codeblock %}
Factory.define :user_absence_example,:class => User do |user|
  user.login "test"
  class << user
    def default_history_records
      [Factory.build(:history_record,:started_at=>Time.now),
       Factory.build(:history_record,:started_at=>Time.now)]
    end
    def default_calendars
      [Factory.build(:calendar),
       Factory.build(:calendar)]
     end
     def default_loggings
      [Factory.build(:logging,:started_at=>1.days.ago),
       Factory.build(:logging,:started_at=>1.days.ago)]
     end
   end
   user.history_records {default_history_records}
   user.calendars {default_calendars}
   user.loggings {default_loggings}
end
{% endcodeblock %}

这个测试数据的构造工厂，可以放在factories.rb文件中，方便其他测试用例使用，也可以直接放到测试文件的before中，仅供本测试文件使用。通过factory的构造，不仅可以为多个测试用例共享同一组测试数据，而且测试代码也简洁明了。

{% codeblock %}
Factory.define :user_absence_example,:class => User do |user|
  user.login "test"
  class << user
    def default_history_records
      [Factory.build(:history_record,:started_at=>Time.now),
       Factory.build(:history_record,:started_at=>Time.now)]
    end
    def default_calendars
      [Factory.build(:calendar),
       Factory.build(:calendar)]
     end
     def default_loggings
      [Factory.build(:logging,:started_at=>1.days.ago),
       Factory.build(:logging,:started_at=>1.days.ago)]
     end
   end
   user.history_records {default_history_records}
   user.calendars {default_calendars}
   user.loggings {default_loggings}
end
{% endcodeblock %}


{% codeblock %}
Factory.define :user_absence_example,:class => User do |user|
  user.login "test"
  class << user
    def default_history_records
      [Factory.build(:history_record,:started_at=>Time.now),
       Factory.build(:history_record,:started_at=>Time.now)]
    end
    def default_calendars
      [Factory.build(:calendar),
       Factory.build(:calendar)]
     end
     def default_loggings
      [Factory.build(:logging,:started_at=>1.days.ago),
       Factory.build(:logging,:started_at=>1.days.ago)]
     end
   end
   user.history_records {default_history_records}
   user.calendars {default_calendars}
   user.loggings {default_loggings}
end
{% endcodeblock %}


{% codeblock %}
Factory.define :user_absence_example,:class => User do |user|
  user.login "test"
  class << user
    def default_history_records
      [Factory.build(:history_record,:started_at=>Time.now),
       Factory.build(:history_record,:started_at=>Time.now)]
    end
    def default_calendars
      [Factory.build(:calendar),
       Factory.build(:calendar)]
     end
     def default_loggings
      [Factory.build(:logging,:started_at=>1.days.ago),
       Factory.build(:logging,:started_at=>1.days.ago)]
     end
   end
   user.history_records {default_history_records}
   user.calendars {default_calendars}
   user.loggings {default_loggings}
end
{% endcodeblock %}


{% codeblock %}
Factory.define :user_absence_example,:class => User do |user|
  user.login "test"
  class << user
    def default_history_records
      [Factory.build(:history_record,:started_at=>Time.now),
       Factory.build(:history_record,:started_at=>Time.now)]
    end
    def default_calendars
      [Factory.build(:calendar),
       Factory.build(:calendar)]
     end
     def default_loggings
      [Factory.build(:logging,:started_at=>1.days.ago),
       Factory.build(:logging,:started_at=>1.days.ago)]
     end
   end
   user.history_records {default_history_records}
   user.calendars {default_calendars}
   user.loggings {default_loggings}
end
{% endcodeblock %}
