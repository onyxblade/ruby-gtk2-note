#Ruby/Gtk2初尝
------

##环境配置
只需在命令行执行 `gem install gtk2` 并稍等片刻。  
需要注意目前gtk2库并不能很好兼容64位ruby，体现在gem安装出错。  

##Hello, World!
这句简短的问候，因K&R的《C语言程序设计》而广泛流传。无论编写多少遍，总能感到初入程序世界的那份欣喜和惊奇。

```ruby
require 'gtk2'

button = Gtk::Button.new("Hello World!")
button.signal_connect("clicked") {
  puts "Hello World!"
}

window = Gtk::Window.new("Example")
window.signal_connect("delete_event") {
  puts "delete event occurred"
  #true
  false
}

window.signal_connect("destroy") {
  puts "destroy event occurred"
  Gtk.main_quit
}

window.border_width = 50
window.add(button)
window.show_all

Gtk.main
```

看上去比以往的HW都复杂？  
没关系，接下来我们将对这个例子进行细致的解构。

##剖析
```ruby
button = Gtk::Button.new("Hello World!")
button.signal_connect("clicked") {
  puts "Hello World!"
}
```
首先，我们新建了一个名为button的按钮，并把它的label属性设为"Hello World!"。接着我们使用了signal_connect方法，将button的点击信号与打印字符串的代码链接了起来。  
Gtk的运作基于信号和回调的，程序运行时，会启动一个主循环（即 `Gtk.main` ），这个主循环在接到信号之前，不会进行任何工作。而当接到一个信号，Gtk即会回调对应这个事件的函数。在这个例子中，当button被点击的信号传给主循环，主循环便会调用 `puts "Hello World!"` ，在命令行打印一行字。  
```ruby
window = Gtk::Window.new("Example")
window.signal_connect("delete_event") {
  puts "delete event occurred"
  #true
  false
}

window.signal_connect("destroy") {
  puts "destroy event occurred"
  Gtk.main_quit
}
```
类似地，我们新建了一个title为"Example"的窗口，并添加了delete_event和destroy两个事件。delete_event会在窗口的关闭键被点击时触发，而这个事件会返回一个值，若为假则会调用destroy事件，这对实现“你确定要关掉它吗？”很有用。而在destroy事件中，简单明了地执行了退出Gtk主循环的命令。  
```ruby
window.border_width = 50
window.add(button)
```
窗口是承载按钮等控件的容器，后者依赖前者以显示。这段代码把button添加进了window。而border_width指的是窗口边界距离内容物的宽度，设成50px让这个窗口看起来有些臃肿，不过这么做只是为了让窗口的标题能正常显示。  
然后，执行 `window.show_all` 让窗口和它包含的按钮显示，这和以下两行是等价的。
```ruby
button.show
window.show
```
最后，使用 `Gtk.main` 启动Gtk主循环，开始等待事件信号。
