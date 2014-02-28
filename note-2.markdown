#使用Glade进行GUI设计
------

##安装Glade
从[Glade官网][1]下载安装文件，Gtk2的设计需要使用Glade 3.8。

##设计器的使用
先来浏览一下左侧的控件栏，嗯，没有什么新奇的。  
那么首先新建一个Window，采取默认名字也就是 `window1` 了。右边可以修改各种属性，暂时只把 `Common - Border width` 改为10。
Gtk控件的排布需要借助容器，在容器栏可以看到 `Horizontal Box`，`Table` 等等容器控件。尝试一下将他们添加到 `window1`，例如 `Table` 会把窗口按照行列分成数格，就像传统的HTML设计方法，接着就可以把按钮等控件放置到每个格子里。  
然而利用表格设计显然太不方便，实际上我们还有更好的选择。  
把刚才的 `table1` 容器删掉，然后在 `window1` 中添加一个 `Fixed` 容器。试试在容器中添加一个按钮 `button1`，可以发现这个按钮会悬浮在你点击的位置。按住Shift键可以对按钮的大小和位置进行调整。再添加一个文本框 `label1`，这个简单的GUI作为示例大致足够了。  
将设计文件保存为 `test.glade`。它的内在大概是这个样子：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<interface>
  <requires lib="gtk+" version="2.24"/>
  <!-- interface-naming-policy project-wide -->
  <object class="GtkWindow" id="window1">
    <property name="can_focus">False</property>
    <property name="border_width">10</property>
    <property name="window_position">center</property>
    <child>
      <object class="GtkFixed" id="fixed1">
        <property name="visible">True</property>
        <property name="can_focus">False</property>
        <child>
          <object class="GtkButton" id="button1">
            <property name="label" translatable="yes">button</property>
            <property name="width_request">100</property>
            <property name="height_request">80</property>
            <property name="visible">True</property>
            <property name="can_focus">True</property>
            <property name="receives_default">True</property>
          </object>
          <packing>
            <property name="x">27</property>
            <property name="y">107</property>
          </packing>
        </child>
        <child>
          <object class="GtkLabel" id="label1">
            <property name="width_request">159</property>
            <property name="height_request">80</property>
            <property name="visible">True</property>
            <property name="can_focus">False</property>
            <property name="label" translatable="yes">label</property>
          </object>
        </child>
      </object>
    </child>
  </object>
</interface>
```
##使用Gtk::Builder绘制GUI
在同目录新建一个ruby源文件，输入以下代码：
```ruby
require 'gtk2'

builder = Gtk::Builder.new
builder.add_from_file('test.glade')
builder['window1'].signal_connect('destroy') { Gtk.main_quit }
builder['window1'].show_all
builder['button1'].signal_connect('clicked') { builder['label1'].label = 'Hello' }

Gtk.main
```
Gtk是利用 `Gtk::Builder` 对glade文件进行读取的，我们会新建一个Builder对象来绘制GUI。利用Builder对象可以直接对控件进行属性和方法的操作，这段代码还是很好理解的。  
运行代码，点击一下按钮，会发现文本框的文字变成了"Hello"。由于我们并未对窗口的大小进行设置，所以实际上窗口大小是依据控件的位置和 `Border width` 决定的。  

##更规范的方法？
在查找资料的过程中，我发现人们一般不会直接用Builder对象操作控件，而是会将控件的信号与函数链接起来。在glade设计器里，对控件可以设置 `Signals` 属性。尝试对 `button1` 的 `clicked` 信号进行设置，将其 `Handler` 设为 `on_button1_clicked`。  
保存，然后运行以下ruby代码：
```ruby
require 'gtk2'

class Builder < Gtk::Builder
  def initialize(file)
    super()
    self.add_from_file(file)
    self['window1'].signal_connect('destroy') { Gtk.main_quit }
    self['window1'].show_all
    self.connect_signals{ |handler| method(handler) }
  end
  def on_button1_clicked
    self['label1'].label = 'Hello'
  end
end

builder = Builder.new('test.glade')
Gtk.main
```
在这个例子中使用了更OO的方法，`connect_signals` 方法把控件的信号和对应的函数都链接起来。比如刚才设置了 `button1` 的 `Handler`，这里就把 `button1` 的点击信号和函数 `on_button1_clicked` 链接了起来。  
虽然应该这才是规范的写法，但是我对此不太理解。为每个信号分配一个特定的函数，是不是降低了视图和程序的分离度？直接使用控件 `signal_connect` 不是很好吗？等待高人解答。
  [1]: http://glade.gnome.org/
