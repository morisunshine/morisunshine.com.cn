---
date: '2014-02-19'
slug: make_your_own_gem
title: "制作自己的Gem"  
category: ruby 
description: "RubyGems是一个方便而强大的Ruby程序包管理器，Ruby的第三方插件是用gem方式来管理，非常容易发布和共享，一个简单的命令就可以安装上第三方的扩展库。特点：能远程安装包，包之间依赖关系的管理，简单可靠的卸载，查询机制，能查询本地和远程服务器的包信息，能保持一个包的不同版本，基于Web的查看接口，能查看你安装的gem的信息。" 
tags: [ruby,译文]
comments: true 
share: true

---

&nbsp;[**什么是Gem？**](#introduce)       
&nbsp;[**第一个Gem**](#first_gem)  
&nbsp;[**包含更多文件**](#include_more_files)  
&nbsp;[**添加可执行文件**](#adding_an_executable)  
&nbsp;[**测试**](#writing_tests)  
&nbsp;[**文档**](#documenting_your_code)  
   

<a id='introduce' name='introduce'> </a>

##什么是Gem?


RubyGems是一个方便而强大的Ruby程序包管理器，Ruby的第三方插件是用gem方式来管理，非常容易发布和共享，一个简单的命令就可以安装上第三方的扩展库。特点：能远程安装包，包之间依赖关系的管理，简单可靠的卸载，查询机制，能查询本地和远程服务器的包信息，能保持一个包的不同版本，基于Web的查看接口，能查看你安装的gem的信息。

---

<a id='first_gem' name='first_gem'> </a>

##第一个Gem

我们要创建一个名叫`moondemo`的gem，首先，就要创建一个名字为`moondemo_yourname`的目录，这个是为了后面的发布，如果你想发布的话，就要检查一下你的gem名字是否已经被人使用了，如果已经被人使用，那就要换个名字了。    
然后这个目录里的基本文件结构应该是这样的。 
 
```ruby
➜  tree  
.  
├── moondemo.gemspec    
├── lib  
│   └── moondemo.rb  

```

gem中的代码被放在`lib`目录中，这里有个约定就是`lib`中必须有个和gem同名的ruby文件，这样当`require 'moondemo'`运行的时候，这个gem就会被加载，这个文件就是负责配置你的gem的代码和API。

```ruby
➜  cat lib/moondemo.rb
class moondemo
  def self.hi
    puts "Hello world!"
  end
end

```

而`.gemspec`文件是定义了这个gem的信息，比如是这个gem的功能，作者等，并且当这个gem发布的时候，会将这些信息显示到这个gem的主页上(就像[jekyll](http://rubygems.org/gems/jekyll))。


```ruby
➜  cat moondemo.gemspec
Gem::Specification.new do |s|
  s.name        = 'moondemo'
  s.version     = '0.0.0'
  s.date        = '2014-02-19'
  s.summary     = "moondemo!"
  s.description = "A simple hello world gem"
  s.authors     = ["sheldon huang"]
  s.email       = 'allenwenzhou@gmail.com'
  s.files       = ["lib/moondemo.rb"]
  s.homepage    =
    'http://rubygems.org/gems/moondemo'
  s.license       = 'MIT'
end

```

这里有很多选项，一看名字就知道他们是要代表什么内容的，如果你还想知道更多就看一些这个[文档](http://guides.rubygems.org/specification-reference/)

当我们创建完成了一个.gemspec，就可以编译出一个gem了，是不是有点小激动啊。但如果想要测试它就必须要在本地安装编译好的gem。

```ruby
➜ gem build moondemo.gemspec
  Successfully built RubyGem
  Name: moondemo
  Version: 0.0.1
  File: moondemo-0.0.1.gem
  
➜ gem install ./moondemo-0.0.1.gem
Successfully installed moondemo-0.0.1
Parsing documentation for moondemo-0.0.1
Installing ri documentation for moondemo-0.0.1
1 gem installed

```

上面这些步骤，只能是在本地已经装好了我们自己的gem，但还没有使用它，
我们需要`require`这个gem然后根据自己定义的方法来使用它。

```ruby

➜ irb
2.0.0-p353 :001 > require 'moondemo'
 => true
2.0.0-p353 :002 > MoonDemo.hi
Hello world!
 => nil
```
现在就可以将你的gem发布到Ruby社区上了，当在发布之前需要将你的帐号安装在电脑上，如果你在RubyGems.org上注册了帐号，那就只需要输入一个命令,再输入自己的密码就可以了

```ruby
➜ curl -u 你的帐号名 https://rubygems.org/api/v1/api_key.yaml > ~/.gem/credentials; chmod 0600 ~/.gem/credentials

Enter host password for user '你的帐号名':

```

一旦你的用户名已经被安装了，就可以直接发布你的gem了。

```ruby
➜ gem push moondemo-0.0.1.gem
Pushing gem to https://rubygems.org...
Successfully registered gem: moondemo (0.0.1)

```
很快的，你的gem就可以被任何人使用了

```ruby
➜ gem install moondemo
Successfully installed moondemo-0.0.1
Parsing documentation for moondemo-0.0.1
1 gem installed

```
用Ruby和RubyGems来分享代码是不是很简单。

---

<a id='include_more_files' name='include_more_files'> </a>

##包含更多文件

我们以后代码当然不会这么简单，如果代码变得非常多了之后，该怎么办呢，当然是要是要将代码分到不同的文件中了。  
比如我们想在刚才的gem中添加根据不同语言来输出不同语言的"Hello world"。  
我们就可以添加一个`Translator`文件，刚才提到过，gem的根文件是负责加载代码的，所以其他的功能的文件就需要放在`lib`中和gem同名的目录中，我们可以这样分:

```bash
➜ tree
.
├── lib
│   ├── moondemo
│   │   └── translator.rb
│   └── moondemo.rb
├── moondemo-0.0.1.gem

```

`Translator`中的内容是:

```ruby
class Translator
  def initialize(language)
    @language = language
  end

  def hi
    case @language
      when "chinese"
        "你好，世界!"
      else
        "Hello world!"
      end
  end
end

```

所以接下来，`moondemo.rb`中需要加载`Translator`:

```ruby
class MoonDemo
  def self.hi(language = "english")
    translator = Translator.new(language)
    translator.hi
  end
end

```

*注意:每次新建了一个目录或者文件，都不要忘记加到.gemspec文件中，就像这样*

```ruby
 s.authors     = ["Sheldon"]
 s.email       = 'allenwenzhou@gmial.com'
 s.files       = ["lib/moondemo.rb","lib/moondemo/translator.rb"]
  
```
*如果没有上面的修改的话，这个新建的目录是不会被加载到已安装的gem里的*

让我们再运行一篇

```ruby
➜ irb -Ilib -rmoondemo
2.0.0-p353 :001 > MoonDemo.hi("english")
 => "Hello world!"
2.0.0-p353 :002 > MoonDemo.hi("chinese")
 => "你好，世界!"
 
```

这里我们使用了一个新的命令行`-Ilib`,通常RubyGems会为你包含了`lib`路径，所以很多时候，我们不需要去考虑配置它们的加载路径，但是，如果你把代码运行在RubyGems的项目之外，你就要自己去配置这些了。

如果你添加了一些新的文件到你的gem中，那么你一定要记住在发布之前将这些文件添加到你的`.gemspec`的`files`数组中。这样很麻烦是么，所以很多人就选择用[Hoe](http://docs.seattlerb.org/hoe/),[Jeweler](https://github.com/technicalpickles/jeweler),[Rake](http://rake.rubyforge.org/classes/Rake/GemPackageTask.html),[Bundler](http://railscasts.com/episodes/245-new-gem-with-bundler)或者用[动态的gemspec](https://github.com/wycats/newgem-template/blob/master/newgem.gemspec)来实现自动化。

添加更多的目录也都是按照上面一样的步骤，我们要将我们的文件结构分布合理，这样对于我们以后的维护和未来开发人员来说就不会是一件头疼的事儿了。

<a id='adding_an_executable' name='adding_an_executable'> </a>

##添加可执行文件

gem除了可以提供Ruby代码库外，还可以在你的可执行文件路径里提供很多可执行可执行文件文件。可能最有名的就是`rake`，
添加一个可执行可执行可执行文件文件其实很简单，你只需要将你的可执行文件放在你的gem的`bin`目录下，然后在将这个文件添加到`.gemspec`文件中`executables`的列表里就可以了，让我们试一下:

```bash
➜  mkdir bin
➜  touch bin/moondemo
➜  chmod a+x bin/moondemo

```
这个可执行文件只需要在开头用[shebang](http://www.catb.org/jargon/html/S/shebang.html)来表明这是用程序来运行的，下面就是这个可执行文件的内容:

```ruby
#!/usr/bin/env ruby

require 'moondemo'
puts MoonDemo.hi(ARGV[0])

```
这个可执行文件的内容很简单，它只是加载了moondemo这个gem，然后在命令行中通过输入一个参数来判断是用哪个国家的语言来说"hello, world"。下面就是运行的例子:

```bash
➜  ruby -Ilib ./bin/moondemo
Hello world!
➜  ruby -Ilib ./bin/moondemo chinese
你好，世界!

```

最后，我们要将这个可执行文件添加到`.gemspec`

```ruby
s.executables << 'moondemo'

```

更新你的gem，上传到官网上，这样你就可以在命令行中有自己的命令了，是不是很帅啊。这里要提醒一下，上传新的gem时，要记得修改`.gemspec`中的版本号。详情看[这儿](http://guides.rubygems.org/patterns/#semantic-versioning)

在看一下用我们自己定义的命令行吧:

```bash

➜  ~  moondemo
Hello world!
➜  ~  moondemo chinese
你好，世界!

```

世界在向我们招手呢。哈哈哈~~~

<a id='writing_tests' name='writing_tests'> </a>

#测试

测试我们的gem是非常重要的，它不仅保证了这个gem是正常的，也保证了别人能知道你的gem是正常的。当我们评价一个gem的时候，很多Ruby开发者会倾向于通过查看测试用例来做为主要依据。

Gems是支持将测试文件添加到程序包中的，所以当gem被下载了之后，我们可以直接运行测试用例。这里有个帮助我们如何在不同框架和解释器下写测试用例的社区叫[GemTesters](http://test.rubygems.org/)
总而言之:去测试我们的Gem吧！啥都别想了！

`Test::Unit`是Ruby的自带测试框架。这里有很多[教程](https://github.com/seattlerb/minitest/blob/master/README.txt)，当然还是有很多其他的测试框架，[RSpec](http://rspec.info/)就是比较有名的一个，是不是迫不及待，让我们测试吧！

我们需要在原来的基础上再添加一些文件，一个名为`Rakefile`的文件和一个名为`test`的目录:

```bash

.
├── Rakefile
├── bin
│   └── moondemo
├── lib
│   ├── moondemo
│   │   └── translator.rb
│   └── moondemo.rb
├── moondemo-0.0.1.gem
├── moondemo.gemspec
├── npm-debug.log
└── test
    └── test_moondemo.rb

```

`Rakefile`文件是为了实现自动化的测试:

```ruby

require 'rake/testtask'

Rake::TestTask.new do |t|
        t.libs << 'test'
end

desc "Run tests"
task :default => :test
~

```

下面就是一个简单的测试用例了:

```ruby
require 'test/unit'
require 'moondemo'

class MoonDemoTest < Test::Unit::TestCase
  def test_english_hello
    assert_equal "Hello world!",
      MoonDemo.hi("english")
  end

  def test_any_hello
    assert_equal "Hello world!",
      MoonDemo.hi("ruby")
  end

  def test_spanish_hello
    assert_equal "你好，世界!",
      MoonDemo.hi("chinese")
  end
end

```

最后，让我们运行这个测试:

```bash
➜  rake test
Run options:

# Running tests:

Finished tests in 0.006151s, 487.7256 tests/s, 487.7256 assertions/s.
3 tests, 3 assertions, 0 failures, 0 errors, 0 skips

```

很好，全部都通过了！

<a id='documenting_your_code' name='documenting_your_code'> </a>

#文档

文档和测试是一样重要的，大部分gem都是用RDoc来生成文档的，这里有很多[教程](http://docs.seattlerb.org/rdoc/RDoc/Markup.html).
很简单，先切换到gem的根目录下，再命令行中输入`rdoc`，你就会发现多出了一个名为`doc`的目录，里面就是我们的文档了，当然你可以自己再进行一些调整。
除了RDoc呢，我们还有另外的选择[YARD](http://yardoc.org/)，当我们发布了gem的时候呢，[RubyDoc.info](http://rubydoc.info/)会根据你的gem自动生成YARDocs，并且它是可以向下兼容RDoc。













