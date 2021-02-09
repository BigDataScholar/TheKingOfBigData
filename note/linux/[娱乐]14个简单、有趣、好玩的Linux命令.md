前言

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信大家对于linux常用的命令一定都不陌生，但是一些简单，好玩，有趣，虽然可能没有实际作用的命令，你又有了解多少呢？话不多说，本期文章为大家带来15个好玩的linux命令，希望大家能够喜欢！

![](https://img-blog.csdnimg.cn/20201220005719743.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## 1、sl 命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;你会看到一辆火车从屏幕右边开往左边……

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然我们需要先安装软件包

```powershell
sudo apt-get install sl
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后运行`sl`即可看到效果

![](https://img-blog.csdnimg.cn/20201219195653525.gif)

## 2、fortune 命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输出一句话，有笑话，名言什么的 (还有唐诗宋词 `sudo apt-get install fortune-zh`)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;软件包安装

```powershell
sudo apt-get install fortune
```
![](https://img-blog.csdnimg.cn/20201213232146512.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这些都是英文的，如果你想看中国的唐诗三百首，则需要再安装：

```powershell
sudo apt install fortune-zh
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;来多执行几次`fortune-zh`，可以看到不同的唐诗:
![](https://img-blog.csdnimg.cn/20201219195906981.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## 3、cowsay 命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用ASCII字符打印牛，羊等动物

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;也是需要先安装软件包

```powershell
sudo apt-get install cowsay
```
![](https://img-blog.csdnimg.cn/20201213232415607.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;        还有一种骚操作是 **cowsay -l 查看其它动物的名字，然后 -f 跟上动物名**，如

![](https://img-blog.csdnimg.cn/20201213232632349.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## 4、cmatrix 命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个就很酷，有《黑客帝国》那种矩阵风格的动画效果

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先安装软件包

```powershell
sudo apt-get install cmatrix
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行`cmatrix`命令即可

![](https://img-blog.csdnimg.cn/2020121919542157.gif)



## 5、figlet 、toilet命令

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;艺术字生成器，由ASCII字符组成，把文本显示成标题栏

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先安装软件包

```powershell
sudo apt-get install figlet
sudo apt-get install toilet
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行

![](https://img-blog.csdnimg.cn/2020121323351117.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

​         toilet 还可以添加颜色

![](https://img-blog.csdnimg.cn/20201213233554817.png)

## 6、 oneko 命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;桌面上出现一直喵星人，跟着你的鼠标跑，你不动了它就睡觉。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;软件包安装

```powershell
sudo apt-get install oneko
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;直接`oneko`命令运行

![](https://img-blog.csdnimg.cn/20201219213215539.gif)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要关掉这个小家伙，直接 `ctrl + c` 即可

## 7、screenfetch命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个命令可以用来显示系统、主题信息

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;软件包安装

```powershell
sudo apt install screenfetch
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;直接运行`screenfetch`命令即可

![](https://img-blog.csdnimg.cn/20201219190156312.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
## 8、linux_logo
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个命令可以用来显示linux版本logo图片及系统信息

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装软件包

```powershell
sudo apt install linuxlogo
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;直接运行`linux_logo`命令即可

![](https://img-blog.csdnimg.cn/20201219190531278.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们可以查看内置的logo列表：

![](https://img-blog.csdnimg.cn/20201219193202853.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;接下来我们开始利用命令在终端循环打印logo:

```powershell
for i in {1..30};do linux_logo -f -L $i;sleep 2;done
```

![](https://img-blog.csdnimg.cn/20201219194928834.gif)


## 9、boxes
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个命令可以实现在输入的文本或者代码周围框上各种ASCII 艺术画，非常有趣！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先安装软件包

```powershell
sudo apt-get install boxes
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在我把一段文字不加任何效果输出是这样的

![](https://img-blog.csdnimg.cn/20201219191704569.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​       现在加上小猫的图案

![](https://img-blog.csdnimg.cn/20201219191759933.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​        亦或者小狗

![](https://img-blog.csdnimg.cn/20201219191841251.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​      小老鼠

![](https://img-blog.csdnimg.cn/20201219192034555.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;更多的效果就不展示了，等待着你们去发现~


## 10、pv
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有时候我们在电影屏幕上看到一些字幕一个个匀速显示出来，像有人在边敲键盘，边显示一样。Linux上的pv命令可以实现这种效果。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先安装软件包

```powershell
sudo apt-get install pv
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里我用英文座右铭运行实验：

![](https://img-blog.csdnimg.cn/20201219194625551.gif)
## 11、aview
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个命令可以将图片以ASCII码格式在终端上显示，有点cool

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装软件包

```powershell
apt-get install aviewm 
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行命令，将一张大象的图片 **elephant.jpg** 进行转码显示

```powershell
asciiview elephant.jpg -driver curses 
```
![](https://img-blog.csdnimg.cn/2020121921434046.png)
## 12、ASCIIquarium
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是一个相当有趣的命令，你可以在你的终端中使用 `ASCIIQuarium` 安全地欣赏海洋的神秘了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;是不是听起来相当神奇，快来尝试下吧

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先需要安装 **Term::Animation** 的 **perl** 模块

```powershell
$ sudo apt-get install libcurses-perl
$ cd /tmp
$ wget http://search.cpan.org/CPAN/authors/id/K/KB/KBAUCOM/Term-Animation-2.4.tar.gz
$ tar -zxvf Term-Animation-2.4.tar.gz
$ cd Term-Animation-2.4/
$ perl Makefile.PL && make && make test
$ sudo make install
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后才开始安装`ASCIIQuarium`

```powershell
$ cd /tmp
$ wget http://www.robobunny.com/projects/asciiquarium/asciiquarium.tar.gz
$ tar -zxvf asciiquarium.tar.gz
$ cd asciiquarium_1.0/
$ sudo cp asciiquarium /usr/local/bin
$ sudo chmod 0755 /usr/local/bin/asciiquarium
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在通过命令`/usr/local/bin/asciiquarium`或者`perl /usr/local/bin/asciiquarium`即可欣赏到 ASCII 水族箱

![](https://img-blog.csdnimg.cn/20201219220505464.gif)

## 13、blessed-contrib

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;想拥有一个高大上仪表盘，假装自己指点江山，纵横捭阖吗，快来试试下面这款！

```powershell
sudo apt-get install npm
sudo apt install nodejs-legacy
git clone https://github.com/yaronn/blessed-contrib.git
cd blessed-contrib
npm install
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后通过命令`node ./examples/dashboard.js`就可以欣赏高大上黑客仪表盘
![](https://img-blog.csdnimg.cn/20201219223546430.gif)

## 14、hollywood
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;高大上仪表盘，假装自己日理万机，宵衣旰食。听起来是不是很不错。Dustin Kirkland 利用一个长途飞行的时间，编写了这个炫酷、有趣但也没什么实际作用的软件。  在其它Linux发行版中，可以通过以下命令安装。 

```powershell
sudo apt-add-repository ppa:hollywood/ppa
sudo apt-get install hollywood
sudo apt-get install byobu
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后通过这个命令`hollywood`即可欣赏到一个炫酷的界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201220003808716.gif)
***

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本期介绍Linux有趣，好玩的命令到这里就结束了，感兴趣的朋友们记得点个关注，在看。后续笔者会为大家更新更多有趣，好玩的内容，敬请期待！你知道的越多，你不知道的也越多，我是Alice，我们下一期见！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

>**文章持续更新，可以微信搜一搜「 猿人菌 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经…期待您的关注!**



<img src="https://img-blog.csdnimg.cn/20210119135940253.png" width = 99% height = 50% />