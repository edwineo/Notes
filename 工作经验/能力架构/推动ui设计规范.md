聊聊UI设计规范：移动端、H5与Web端

### 1. UI设计规范是什么

UI设计规范，是指由UI设计师制定的，对配色、字体、元素间距、控件尺寸、页面布局等可复用设计规则的提炼和约束，用于指导后续的设计工作。制作UI设计规范是一件看起来非常容易的事情，一个psd文件、一个pdf，甚至一个ppt文件都可以作为设计规范的载体。
然而UI规范的逐渐完善、坚持在产品中推行，以及整个团队切实意识到规范的必要性，则是需要UI设计师多年积累的一个过程。

### 2. 设计规范的价值

UI设计规范是一份在团队协作中，对降低沟通成本、提高设计效率、保持产品或品牌设计风格一致而言非常重要的文档，团队越大，规范的重要性就越发凸显。对开发而言，可以便于开发同学准确地还原样式、增强控件的复用性。对设计师团队而言，便于向新介入项目的同事介绍项目概况、帮助其迅速上手项目。对公司品牌而言，则有助于向用户传达统一的品牌形象。
当然，严格而精细的设计规范在不注重设计的团队中坚持推行却并不容易，需求的变化速度、前端同学的脾气、领导的审美水平……等等很多因素会造成一套UI规范的推行无疾而终。不过，在规模过小的团队中，甚至设计师只有一个人的情况下，UI设计规范是否必要、甚至是否会影响工作效率，则确实需要设计师自己进行斟酌。

### 3. 设计规范的制订时机

规范的制定是在设计之前进行还是设计之后进行，不同团队有不同的看法。有主张先制定规范的，有主张同时进行的，有主张在整套UI设计做好后再进行规范的整理的（例如网易）。个人认为还是在整套UI基本成型后再进行更为合理，UI没成型前就制定规范的话，一来容易欠考虑而漏掉很多需要约束的东西，二来过早地被规范约束的设计很可能会影响创意的发挥。无论如何选择规范和设计的先后，规范都是一个逐渐完善和积累的过程，在初稿形成后，不断进行归纳、微调和后续补充是一份优秀规范积淀的必经之路。

### 4. 设计规范的内容

不同终端的产品所需的UI设计规范大体内容相同（如色彩、控件、文字排版规范），但在此之外仍然存在一定的差别。
（1）移动端的UI设计规范（主要指原生应用）：
A. 需要指明尺寸适配的屏幕是2x屏还是3x屏（1x屏现在已经很少见了）。
B. 切图要注意点击区域，可以连同点击区域一起切图，避免点击区域过小（对iOS，所有点击区域不能小于88px）。
C. Android开发中需要.9.png格式的特殊切图，即对普通png扩大画布大小，上下左右四周各留1像素，并将最外围的一圈像素涂成纯黑。
D. 考虑手势的效果。
（2）H5页面的UI设计规范：
A. 有别于原生应用，需要考虑多尺寸屏幕适配的布局规范和响应性问题，例如布局变化的临界点、栅格设计等。
B. 手势操作设计应避免使用与浏览器本身有冲突的操作，减少手势操作。
C. 样式尽量与原生控件风格一致。
（3）Web端的UI设计规范：
主要区别在于需要指定控件的悬停效果，且无需考虑复杂的手势效果。另外，同样需要考虑不同分辨率下的布局规范。

### 5. 制定流程

# 参考

[如何建立UI设计规范](https://juejin.cn/post/6844903753516646413)

[聊聊UI设计规范：移动端、H5与Web端](https://static.kancloud.cn/chandler/user-ex-and-programming/1019685)