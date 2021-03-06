# Mobile webview acceleration

## Abstract
无论是安卓还是iOS系统，在移动端的Native App上打开web页面、web应用均是通过创建webview的方式来进行的，webview的性能直接影响到web页面打开的性能，webview的易开发和易维护也直接影响到移动端开发和变更的效率。大量的实际应用中，可以发现移动端上的webview的性能存在诸多不足，一定程度上已经成为了Native App上影响用户web体验的关键因素之一，而且目前并没有全面、通用的解决方案。

## issues
Native App上使用webview打开web页面和web应用，主要存在如下的一些性能、开发维护上的不足或问题：

* 自身性能表现差，存在瓶颈：
	 - 移动端创建一个webview的过程很慢，耗时长，造成web打开体验的不流畅，等待动画时间过长；
	 - 移动端上的webview的使用上，默认每一个web页面的打开都需要新创建一个webview，没有webview的复用机制，特别对于web页面多的应用，需要频繁的创建和删除，资源开销大，性能差；
	 - 创建后的webview，如果不删除该webview，那么它里面展现的内容不能或者不易进行清除和替换，同时强行替换还容易出现显示异常；
	 -  目前移动端的webview无法在后台进行创建，创建过程中如果不注意处理，webview就会占用Native App上的UI线程，会造成页面的卡顿，对开发者带去难题；
	 - 已经加载页面的webview无法实现后台待命并在需要读取同一页面的时候直接调出而不再重新读取页面数据，目前每次web页面读取时，无论用户是否已读，无论页面是否有缓存，都是需要重新加载页面；
	 - 受移动操作系统的限制，webview在打开web页面的动画过程中，无法进行同步渲染，而只能显示动画，没法通过打开过程中提前渲染来使动画时间减少、提高视觉体验；
 
* 硬件资源占用多、不可复用：
	 - 在硬件绘制方面，每增加一个新的webview在屏幕上显示，都需要在移动终端上进行硬件资源的申请和释放，独占的特性使得不能做到对硬件资源充分和有效的复用，频繁的申请和释放极大的影响了性能和浪费了资源；

* 开发维护不方便，开发受限：
	 - 移动端上的webview不能实现用户更快看到内容展现在屏幕上，需要等页面及其内容完整渲染完之后，才能开始显示到屏幕上，制约了性能改善和开发者；
	 - 移动端上的webview自身缺乏多实例的管理，对端的依赖较多，制约了开发者的开发；
	 - 移动端webview缺乏整套对外开放的通用的下载、读取及缓存的通用能力及相应的管理和维护能力，Native App上为了实现下载、读取、缓存管理，进行了各种不同的开发实现，多是临时、短线方案，给端上持续开发、更新带去不少挑战，而这随着页面元素内容及其操作的增多将更加严重；
	 - 移动端上的webview无法获取或提供页面当前屏幕部分完全渲染展示在屏幕上的时间点，对于开发评测带去挑战。
	 - 随着系统版本不同，webview自身版本也会不同（比如回调的修改和增删），在兼容性方面对开发者具有很大的挑战。

* 缺少统一的规范，差异较多：
	 - 目前W3C中不存在针对webview使用的的标准规范，webview的定义、开发、维护都在操作系统提供方和浏览器引擎的提供方。
	 - 
		
## solutions
面对上述罗列的移动端webview使用过程中的各种性能、开发和维护方面的不足或问题，下面重点针对其中主要的一些问题，给出解决方案：

* 移动端在web页面打开之前实现webview的预先创建，解决webview创建慢的问题，同时webview创建可以实现在后台进行，不影响移动端上的UI效果。
* 移动端实现webview的复用，解决频繁创建和删除的问题，可以按照实现分为三个层次：
	 - Level 1：打开web页面前预先在移动端创建一个webview，在页面退出时删除这个webview，并创建一下各新的一个新的webview待用。
	![图片](http://bos.nj.bpc.baidu.com/v1/agroup/4c34118b012ef4ef19c0616e8fc725ad664b8583)
	 - Level 2：移动端上的webview只需要创建一次，退出页面后，新的页面打开时可以直接复用该webview，在退出页面时清除webview中的原页面内容，但不删除webview，然后填充新页面，或者直接支持在webview里新页面替换已有页面。
	![图片](http://bos.nj.bpc.baidu.com/v1/agroup/586abe4cdffb684159831fe54784c171b33bb1ed)
	 - Level 3：之处多个webview同时存在、协同使用，webview实现多实例的管理，具体来说单个webview可以支持复用，多个webview均可加载页面，同时多个webview之间可以无缝的切换。
	![图片](http://bos.nj.bpc.baidu.com/v1/agroup/09b0856948d1935c3ec474eae13731702f518e85)
* 移动端webview添加到屏幕显示不独占硬件资源，支持并行化的渲染，充分复用硬件资源，并且支持对移动端硬件资源的更好管理，避免频繁和重复申请和释放。
* 移动端webview支持对渲染方式的配置，可以支持内容的优先渲染和展示到屏幕上
* 移动端webview提供对端开放的通用的下载、读取、缓存管理能力。

## use case
当今的移动互联网进入内容分发时代后，新闻资讯类、社交分享类应用大量出现，在native App上，其中图文、图集、广告、第三方页面等web页和应用均使用webview打开。通过webview预先创建和复用将webview的耗时和资源频繁占用和使用降到尽可能低的程度，可以较大的提高web页面和应用打开的速度和体验，同时也能提高整个Native App的开发便利性和运行效率。
